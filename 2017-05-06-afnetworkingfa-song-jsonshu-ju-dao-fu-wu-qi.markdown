---
layout: post
title: "AFNetWorking发送JSON数据到服务器"
date: 2015-11-11 15:58:48 +0800
comments: true
categories: Dev
tags: [IOS]
---


<!--more-->
一般来说，客户端都是发送字典形式的数据到服务器来获得资源，而服务器一般以`JSON`格式数据返回。
现在需求是客户端发起POST请求，发送`JSON`格式的数据到服务器获取资源。
用`Python`脚本模拟如下
 
```python
#!/usr/bin/python
# coding: utf-8

import requests
import json

post_url = "http://api.tuicool.com/api/login.json"
headers = {
    "User-Agent": "iOS/iphone5/2.13.1",
    "Host": "api.tuicool.com",
    "Accept-Language": "zh-CN",
    "Authorization": "Basic MC4wLjAuMDp0dWljb29s",
    "Content-Type": "application/json"
}

form_data = {
    "email": "xx@qq.com",
    "password": "qwertasdfg"
}

json_str = json.dumps(form_data)
print json_str

s = requests.session()
s.headers = headers
response = s.post(post_url, data=json_str, headers=headers)
print response.text
```


`Objective-C`中`NSURLRequest`中有一个`HTTPBody`刚好满足这个需求。
 
```objective-c
- (void)loginWithEmail:(NSString *)email Password:(NSString *)password andBlock:(void (^)(id responseObject, NSError *error))block{
    static NSString *loginURL = @"http://api.tuicool.com/api/login.json";
    //转化为json字符串
    NSDictionary *params = [NSDictionary dictionaryWithObjectsAndKeys:email, @"email", password, @"password", nil];
    NSData *data = [NSJSONSerialization dataWithJSONObject:params options:NSJSONWritingPrettyPrinted error:nil];
    [self defaultHTTPHeader];
    
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:loginURL]];
    [request setAllHTTPHeaderFields:[self.requestSerializer HTTPRequestHeaders]];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    request.HTTPBody = data;
    request.HTTPMethod = @"POST";
 
    AFHTTPRequestOperation *operation = [self HTTPRequestOperationWithRequest:request success:^(AFHTTPRequestOperation * _Nonnull operation, id  _Nonnull responseObject) {
        NSLog(@"response = %@", responseObject);
    } failure:^(AFHTTPRequestOperation * _Nonnull operation, NSError * _Nonnull error) {
        NSLog(@"error=%@", error);
        block(operation, error);
    }];
    
    [self.operationQueue addOperation:operation];
}
```





