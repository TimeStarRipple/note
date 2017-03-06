# Servrlt

## jsp
pageContext javax.servlet.jsp.PageContext
request javax.servlet.http.HttpServletRequest
response javax.servlet.http.HttpServletResponse
session javax.servlet.http.HttpSession
application javax.servlet.ServletContext
config javax.serlvet.ServletConfig
exception java.lang.Throwable
page java.lang.Object
out javax.servlet.jsp.JspWriter
作用：
1、pageContext 表示页容器 EL表达式、 标签 、上传
2、request 服务器端取得客户端的信息：头信息 、Cookie 、请求参数 ，最大用处在MVC设计模式上
3、response 服务器端回应客户端信息：Cookie、重定向
4、session 表示每一个用户，用于登录验证上
5、application 表示整个服务器
6、config 取得初始化参数，初始化参数在web.xml文件中配置
7、exception 表示的是错误页的处理操作
8、page 如同this一样，代表整个jsp页面自身
9、out 输出 ，但是尽量使用表达式输出