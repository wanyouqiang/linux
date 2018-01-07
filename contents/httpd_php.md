# Apache Http Server与PHP

Apache Http Server是一个Web服务器软件，PHP是使用C实现的动态脚本语言解析器，Apache Http Server响应Web请求，Apache Http Server将PHP做为一个模块来执行，在类Unix操作系统中，Apache的模块都是以.so结尾的文件位于modules目录下，要想整合Apache Http Server与PHP，在编译安装Apache Http Server时要指定--enable-modules=so(允许加载模块)，在编译PHP时要指定--with-apxs2=/usr/local/httpd/bin/apxs,安装相应模块到modules目录，并在配置文件中添加相应配置。apxs(APache eXtenSion tool),功能是使用mod_so中的LoadModule指令，加载指定模块到Apache,要求Apache要打开So模块
