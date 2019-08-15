> 对于现在刚进入 iOS 开发的程序猿来说，可能对于 NSURLConnection 这个古董鲜为人知，在 iOS9.0 之后，苹果官方已经不推荐使用了，但是学无止境，大家可以看看了解一下。针对苹果目前推崇的 NSURLSession，请关注后续更新。

在使用 NSURLConnection 进行网络请求的时候有很多种方法，包括有：GET、POST、OPTIONS、HEAD、PUT、DELETE、TRACE、CONNECT、PATCH，其中最常用的就是 **GET** 请求和 **POST** 请求。

![NSURLConnection 发送请求](http://upload-images.jianshu.io/upload_images/2997426-a13efab8a899e422.png)

# GET 请求与 POST 请求的区别：

- 主要区别在数据传递上
- GET 在请求 URL 后面以 ？形式跟上发给服务器的参数，多个参数之间用 & 隔开，并且 URL 的长度有限制，通常不超过 1KB
- POST 发给服务器的参数全部放在请求体中，理论上没有限制
- 如何选择：
	- 如果要传递大量数据，比如文件上传，只能用 POST
	- GET 的安全性差，如果传递机密信息，使用 POST
	- 如果仅仅是数据查询，建议使用 GET
	- 如果是增删改使用 POST

# 网络响应的状态

- 200 OK 请求成功
- 400 Bad Request 客户端请求语法错误，服务器无法解析
- 404 Not Found 服务器找不到资源
- 500 Internal Server Error 服务器内部错误，无法完成请求

# 发送同步 GET 请求

1. 确定请求路径（使用 GET 请求时，请求的参数放在路径中）

	```objective-c
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
  ```
  
2. 创建请求对象（默认情况下就是 GET 请求）

	```objective-c
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
  ```
  
3. 发送同步请求
	
	```objective-c
    NSURLResponse *response = nil;
    NSError *error = nil;
    /**
     第一个参数：请求对象
     第二个参数：响应(头)
     第三个参数：错误信息
     返回值：响应体
     */
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
  ```
  
4. 解析服务器返回的数据

	```objective-c
    NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
  ```

# 发送异步 GET 请求

1. 确定请求路径（使用 GET 请求时，请求的参数放在路径中）

	```objective-c
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
  ```
  
2. 创建请求对象（默认情况下就是 GET 请求）

	```objective-c
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
  ```
  
3. 发送异步请求
4. 解析服务器返回的数据

	```objective-c
    /**
     第一个参数：请求对象
     第二个参数：操作队列->(线程) 决定completionHandler回调在哪个线程中处理（主队列->主线程）
     第三个参数：completionHandler回调 完成之后的回调
                response:响应头信息
                data：响应体信息
                connectionError：错误信息
     */
    [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        //4.解析服务器返回的数据
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
	```

# 使用代理发送异步 GET 请求

1. 确定请求路径（使用 GET 请求时，请求的参数放在路径中）

	```objective-c
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520&pwd=520&type=JSON"];
  ```
  
2. 创建请求对象（默认情况下就是 GET 请求）

	```objective-c
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
	```
  
3. 设置请求代理，遵守协议

	```objective-c
    NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self];
  ```
  

***注意：*** 协议使用的是`NSURLConnectionDataDelegate`
		
***补充：*** 以上设置代理会立即发送请求，如要控制是否立即发送，可以使用以下方法并调用 start 手动发送网络请求

```objective-c
NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];
[connection start];
```

4. 实现代理方法
	- 设置代理发送 GET 请求时，主要使用的是以下四个代理方法
	
	```objective-c
    -(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    		NSLog(@"接收到响应 --- didReceiveResponse");
    }
    //可能调用多次，数据分段传输
    -(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
        	NSLog(@"接收到服务器返回的数据 --- %zd", data.length);
        	[self.resultData appendData:data];
    }
    -(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
        	NSLog(@"请求失败 --- didFailWithError");
    }
    -(void)connectionDidFinishLoading:(NSURLConnection *)connection {
        	NSLog(@"完成响应之后 --- connectionDidFinishLoading");
        
        	//解析服务器返回的数据
        	NSLog(@"%@", [[NSString alloc] initWithData:self.resultData encoding:NSUTF8StringEncoding]);
    }
  ```

> 在 POST 请求中，也有同步 POST 请求、异步 POST 请求、设置代理请求等方式，使用方法和 GET 请求一致，这里就不一一赘述，仅以异步 POST 请求为例，说明其中的区别与注意点。

# 发送异步 POST 请求

1. 确定请求路径（请求参数不能放到路径中）

	```objective-c
	NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];
  ```
  
2. 创建可变请求对象（默认为 GET 请求）

	```objective-c
    NSMutableURLRequest *requestM = [NSMutableURLRequest requestWithURL:url];	
  ```
	- 修改请求方式为 POST
	
		```objective-c
	    requestM.HTTPMethod = @"POST";
    ```
    
	- 设置请求体（请求参数放到请求体中）
	
		```objective-c
	    requestM.HTTPBody = [@"username=520&pwd=520&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];
  	```
  
3. 发送异步请求
4. 解析服务器返回的数据

	```objective-c
    [NSURLConnection sendAsynchronousRequest:requestM queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        
        //4.解析服务器返回的数据
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
  ```

# 拓展

- ① `中文乱码转码问题`
	- 在以上发送请求的过程中，如果请求路径中存在中文，需要在发送请求之前对中文进行转码，否则会请求失败
	- 在我们日常使用浏览器的时候，浏览器会自动对 URL 进行转码处理
	
		```objective-c
    	//如果请求路径中有中文，需要对中文进行转码操作
        NSString *string = @"http://120.25.226.186:32812/login2?username=哈哈哈&pwd=520&type=JSON";
        //转码
        string = [string stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        NSURL *url = [NSURL URLWithString:string];
    ```

- ② `NSURLConneection 与 Runloop 的联系`
	- 设置代理的方法中，如果立即发送请求，那么内部会自动将 connection 作为一个 source 添加到 runloop 中
	- 设置代理的方法中，如果设置为 NO 不立即发送请求，那么内部不会添加 connection 到 runloop 中，需调用 start 方法
	- 如果当前 connection 未被添加到 runloop 中，那么 start 内部会把当前的 connection 添加到 runloop 并设置模式为默认，如果 runloop 不存在，那么会主动创建
