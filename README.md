工程目录示意及说明<br>
----

frontend/<br>
>>frontend/addons/:用于每个工程自定义的view来满足特定的交互需求。同样通过自定义的manager类将view注册到对应的服务上。使用时要在工程配置文件中将自定义的manager类加入ADDON_MANAGERS参数中。<br>
>>frontend/util/:支撑实现相关服务。<br>
>>frontend/frontend_server.py:程序的主入口，可以根据指定的参数来启动对应的服务，对于每个服务的APP类一般都包含了app、celery_app、appbuilder。appbuilder在初始化时会将入ADDON_MANAGERS参数中包含的自定义manager类加载进来。<br>
>>frontend/frontend_tool.py:用于提供flask的命令行接口，便于快捷的交互测试。<br>
>>***/:自定义的各个服务，如zhidao、kg等。<br>
>>BUILD:用于blade构建，注意每创建一项APP服务需要在其中添加对应的依赖项


实现方法
-----

(1)总体框架:frontend目录下的工程都是基于flask_appbuilder实现并通过gunicorn拉起服务。整套代码使用时采取blade构建，superivisor运行和管理。<br>
(2)代码运行流程:frontend_server作为主入口会根据传递进来的module_path参数选择加载的app实例并将该app交由gunicorn控制运行，这里gunicorn通过frontend/util/gunicorn_http_server.py实现。搭建新服务时需要重点关注app实例的实现<br>
(3)基于flask_appbuilder的应用服务的构建可以参考参考https://flask-appbuilder.readthedocs.io/en/latest/。<br>
(4)Appbuilder的初始化过程可以参考源码https://github.com/dpgaspar/Flask-AppBuilder/blob/7b796071bae32c5bac30f1d2f2b29b42814b7709/flask_appbuilder/base.py。

调试及部署
-----

(1)blade构建:进入frontend/下，执行blade build,相关问题可以参见lucis工程的README。<br>
(2)调试:<br>
	* 初始化环境:主要是数据库的表，app/model.py为空且不使用ldap可以忽略,进入blade构建后的路径,FLASK_APPLICATION_CONFIG="somepath/test_config.py" FLASK_APP=frontend.somepath.app ./flask_tool fab create-db<br>
	* 启动http server:进入blade构建后的路径,FLASK_APPLICATION_CONFIG="somepath_test_config.py"./frontend_server --module_path="frontend.somepath.app" --workers_num=1。根据输出的访问路径，选择对应的服务在浏览器中打开即可进行测试。<br>
(3)服务部署:在${JUMBO_ROOT}/etc/supervisor.d/目录下编写ini配置文件,执行supervisorctl update让配置生效并启动服务。

    
spider_proxy用途：
-----

(1)为pyspider爬虫中使用self.crawl()发起抓取请求失败但可以通过requests.get()抓取的url,搭建一个中间代理。<br>
(2)请求用法参见http://10.134.13.101:5000/debug/forbes项目，将原url作为参数发送至该代理，该代理会将原请求的url通过requests.get()获得结果并返回response.text。<br>

