## nginx 到uwsgi 的大概流程
1. nginx 接受到请求，如果match到location，location被uwsgi_pass
2. nginx 按wsgi协议，构建请求头、读取body，通过tcp 连接发生给后端的uwsgi，并设置用于返回给用户数据的回调方法
3. uwsgi 收到请求，处理请求，返回数据
4. nginx 将数据返回给用户


## nginx 读取请求的body数据
nginx发送数据到uwsgi：首先nginx会判断用户是否设置client_max_body_size指令，如果设置了，则会用该值来和content-length进行比较，如果发送的包体超过了设置的值，则nginx返回413包体过大的错误。如果包体在给定范围内，则nginx会根据proxy_request_buffering是否开启，来决定是否先缓存客户端发来的请求。如果关闭了proxy_request_buffering(默认开启)，则nginx是接收到一部分数据就直接发送给uwsgi；如果开启了proxy_request_buffering，则nginx是是把所有的数据都读取到之后，再发送给uwsgi，如果body过大，则可能需要把body先存入临时文件中。


## uwsgi 返回数据到nginx
  uwsgi返回数据到nginx：如果uwsgi_buffering开启(默认开启)，nginx会尽可能缓存uwsgi发送来的body，缓冲区的大小由uwsgi_buffers和uwsgi_buffer_size两个指令设置的缓冲区之和；如果还是存不下，则需要写入临时文件，临时文件的大小由uwsgi_max_temp_size和uwsgi_temp_file_write_size决定；如果关闭，则数据同步的发送给浏览器，每次最多缓存uwsgi_buffer_size的数据，可以从upstream->recv()方法看出，如果满了，会导致uwsgi无法发送数据而阻塞。


## nginx 设置client_max_body_size最大合理的大小；python按照具体接口的业务，读一部分，超过大小就扔掉
在工作中，一般是保持proxy_request_buffering和uwsgi_buffering默认开启的设置，这样才能充分利用nginx高并发的特性，不会让一个连接长时间占用后端的web application；另外如果要限制请求body的大小，如上传图片，一般是nginx开到最大合理的大小，然后python按照具体接口的业务，读一部分，超过大小就扔掉。因为nginx可能需要限制多个请求body的大小，所以一般设置一个最大值，然后在web应用中根据需求来加以限制。

<font color=#ff0000 size=72>要设置颜色的文字</font>

