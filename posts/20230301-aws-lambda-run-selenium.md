# 如何使用 AWS Lambda 运行 selenium

> 发布日期：2023-03-01 21:09:00

借助 AWS Lambda 运行 selenium 来爬取网络数据。

## 简介

与手动从网站收集数据相比，爬虫可以为我们节省很多时间，对于爬虫的每次请求而言，这相当于 AWS Lambda 的每次函数的运行。

AWS Lambda 是一种将脚本部署到云的简单且价格低廉的服务，如果我们要实现在 AWS Lambda 上运行 selenium 实现数据的爬取，我们需要解决如何在 AWS Lambda 函数中安装 Chrome 浏览器？

同时，AWS Lambda 的主要限制是超时限制，即 15 分钟，部署包不能超过 250 MB（但使用容器最多可接受10 GB）。因此使用容器化的部署方式便成了不二之选。

相对于我们单独运行一个 [Selenium Grid](https://www.selenium.dev/documentation/grid/) 来进行 Chrome 远程调用，使用 Lambda 的方式运行有以下好处。

- 无需担心 Chrome 在负载高的情况下崩溃，因此每次 Lambda 的执行都会启动一个新的实例。
- 更容易进行大量的并发执行。
- 无需担心服务崩溃。

## 程序部署

本文重点在于如何使用 Docker 部署 Selenium 到 AWS Lambda，因此在看之前可以阅读[在 AWS lambda 上部署 docker 应用](https://blog.csdn.net/crown_0726/article/details/125712948)。

#### 创建项目结构

创建如下的项目结构，`app.py` 是 Lambda 程序的主要入口，我们通过 Dockerfile 来构建一个集成了 Selenium，Chrome 的运行环境。

```
docker-selenium-test
  - app.py
  - Dockerfile
  - requirements.txt
  - README.md
```

#### Dockerfile 配置

这也是核心的配置。

```dockerfile
FROM public.ecr.aws/lambda/python:3.8 as build
# 下载 Linux 版本的 Chrome 和对应的驱动文件
RUN yum install -y unzip && \
    curl -Lo "/tmp/chromedriver.zip" "https://chromedriver.storage.googleapis.com/107.0.5304.62/chromedriver_linux64.zip" && \
    curl -Lo "/tmp/chrome-linux.zip" "https://www.googleapis.com/download/storage/v1/b/chromium-browser-snapshots/o/Linux_x64%2F1047731%2Fchrome-linux.zip?alt=media" && \
    unzip /tmp/chromedriver.zip -d /opt/ && \
    unzip /tmp/chrome-linux.zip -d /opt/

FROM public.ecr.aws/lambda/python:3.8

# 安装 Chrome 运行需要的依赖
RUN yum install atk cups-libs gtk3 libXcomposite alsa-lib \
    libXcursor libXdamage libXext libXi libXrandr libXScrnSaver \
    libXtst pango at-spi2-atk libXt xorg-x11-server-Xvfb \
    xorg-x11-xauth dbus-glib dbus-glib-devel -y

# 将 pre-build 阶段构建的 Chrome 和对应的驱动文件复制到运行环境
COPY --from=build /opt/chrome-linux /opt/chrome
COPY --from=build /opt/chromedriver /opt/

# 运行代码配置
COPY . ${LAMBDA_TASK_ROOT}
WORKDIR ${LAMBDA_TASK_ROOT}
RUN pip install -r requirements.txt

CMD [ "app.handler" ]
```

#### app.py 代码实现

这一步主要是程序测试代码的实现。

```python
import sys
from tempfile import mkdtemp
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.service import Service

CHROMIUM_PATH = '/opt/chrome/chrome'
CHROMEDRIVER_PATH = '/opt/chromedriver'


def handler(event, context):
    print(event)
    # print(context)
    print('Hello from AWS Lambda using Python' + sys.version + '!')

    options = webdriver.ChromeOptions()
    options.binary_location = CHROMIUM_PATH
    options.add_argument("--headless")  # Hide the GUI
    options.add_argument("--no-sandbox")  # No protection needed
    options.add_argument("--window-size=1280x1696")  # Setup a fixed screens size
    options.add_argument("--single-process")  # Lambda only give us only one CPU
    options.add_argument("--no-zygote")  # Don't create zygote processes because Lambda give us only one CPU
    options.add_argument("--disable-dev-shm-usage")  # Create temporary folder for shared memory files
    options.add_argument("--disable-dev-tools")  # Disable Chrome dev tools
    options.add_argument(f"--user-data-dir={mkdtemp()}")  # Create temporary folder to user data
    options.add_argument(f"--data-path={mkdtemp()}")  # Create temporary folder to browser data
    options.add_argument(f"--disk-cache-dir={mkdtemp()}")  # Create temporary folder to cache

    service = Service(CHROMEDRIVER_PATH)
    driver = webdriver.Chrome(service=service, options=options)

    # 开始 ----------
    driver.get("https://example.com")
    header = driver.find_element(By.CSS_SELECTOR, "h1")
    text = header.text
    driver.close()
    driver.quit()
    response = {
        "statusCode": 200,
        "body": f"the header content is {text}",
    }
    print(response)

    return {}
```

在打包好 Docker 镜像之后，推荐使用 AWS ECR 进行镜像存储和管理。具体也可参考文章[在 AWS lambda 上部署 docker 应用](https://blog.csdn.net/crown_0726/article/details/125712948)。

## 进一步实现文件下载

除了获取网页内容之外，我们同时也可以通过 Chrome 进行文件的下载。Selenium 驱动 Chrome 进行文件下载重要的是指定下载文件的路径。

通常指定的方式比较简单如下：

```python
options.add_experimental_option('prefs', {
    "download.default_directory": tmp_dir,  # Change default directory for downloads
    "download.prompt_for_download": False,  # To auto download the file
    "download.directory_upgrade": True,
    "plugins.always_open_pdf_externally": True  # It will not show PDF directly in chrome
})
```

但在无头浏览器模式下这是行不通的，我们需要下面的配置方式：

```python
# 先得到 driver
# Headless 模式下要这种设置才能指定下载路径
driver.command_executor._commands["send_command"] = ("POST", '/session/$sessionId/chromium/send_command')
params = {'cmd': 'Page.setDownloadBehavior', 'params': {'behavior': 'allow', 'downloadPath': tmp_dir}}
driver.execute("send_command", params)
```

这样就可以实现文件下载了。

## 总结

这是 AWS Lambda 和 Selenium 使用的一个非常具体的例子，但我希望它能说明这些技术的潜力。不仅仅是数据的获取，同时我们也可以借助这项技术来进行端到端的测试，或通过构建 API Gateway 的方式进行同步的方法调用等。

## 参考文档
[1] How To Use Selenium To Web-Scrape on AWS Lambda https://cheesecakelabs.com/blog/selenium-scraper-aws-lambda/
[2] lambda/python https://gallery.ecr.aws/lambda/python
[3] Python – Selenium Download a File in Headless Mode https://www.onlinetutorialspoint.com/selenium/python-selenium-download-a-file-in-headless-mode.html
[4] Selenium give file name when downloading https://stackoverflow.com/questions/34548041/selenium-give-file-name-when-downloading
