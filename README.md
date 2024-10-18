# MyObsidianNote
This is my own study note

The content may be a little silly,if you have any suggestions,you can contact me by email.

**email:** 837606776@qq.com
这段代码实现了一个简单的代理服务器，主要功能包括初始化套接字、监听客户端请求、处理请求并将缓存数据读写到文件中。

首先，在 main 函数中，程序通过调用 InitSocket函数来初始化套接字环境。如果初始化失败，程序会输出错误信息并返回 -1 终止运行。初始化成功后，程序会读取缓存文件 

cache02.txt

 的内容到 Cache 结构体数组中。

接下来，程序进入一个无限循环，不断监听客户端请求。通过 accept函数，程序接收客户端的连接请求，并创建一个新的 ProxyParam 对象来存储客户端的套接字。如果对象创建失败，程序会继续监听下一个请求。成功创建对象后，程序会启动一个新线程来处理客户端请求，线程函数为 ProxyThread，并将 ProxyParam 对象作为参数传递给线程函数。线程创建后，程序会立即关闭线程句柄，并将缓存数据写入文件 

cache02.txt

。

在 InitSocket 函数中，程序首先加载套接字库，并检查加载是否成功。然后，程序创建一个 TCP 套接字，并将其绑定到本地地址 `127.0.0.1` 和指定的端口 ProxyPort。最后，程序调用 listen 函数使套接字进入监听状态，等待客户端连接请求。

[`write_to_file`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A116%2C%22character%22%3A4%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A716%2C%22character%22%3A5%7D%7D%5D%2C%22c3a96c15-06d2-4e05-b223-f97e532c9967%22%5D "Go to definition") 函数用于将缓存数据写入指定文件。函数首先打开文件，如果文件打开成功，程序会逐行写入缓存数据，包括 URL、Host、Last Modified、Status 和 Buffer 等信息。写入完成后，程序关闭文件并输出成功信息。

[`read_from_file`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A88%2C%22character%22%3A4%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A739%2C%22character%22%3A5%7D%7D%5D%2C%22c3a96c15-06d2-4e05-b223-f97e532c9967%22%5D "Go to definition") 函数用于从指定文件读取缓存数据。函数首先尝试打开文件，如果文件打开失败，程序会输出错误信息并返回。文件打开成功后，程序逐行读取文件内容，并解析每一行的数据，将其存储到 [`Cache`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A88%2C%22character%22%3A33%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A716%2C%22character%22%3A52%7D%7D%2C%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2FC%3A%2FUsers%2Fquben%2FDesktop%2Fcnet%2Flab1.cpp%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A739%2C%22character%22%3A53%7D%7D%5D%2C%22c3a96c15-06d2-4e05-b223-f97e532c9967%22%5D "Go to definition") 结构体数组中。读取完成后，程序关闭文件。

总体来说，这段代码实现了一个基本的代理服务器功能，包括初始化套接字、监听客户端请求、处理请求以及缓存数据的读写操作。