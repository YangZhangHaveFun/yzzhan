---
title : "对生成器和协程的理解"
date: 2019-01-16T01:37:56+08:00
lastmod: 2019-01-16T01:37:56+08:00
draft: false
tags: ["Python", "generator","coroutine"]
categories: ["Advanced Python Theory"]
author: "Yang Zhang"

---
## 迭代器和迭代协议
我们知道实现下标的操作是基于__getitem__这个协议


1. 可迭代类型

2. 迭代器
#

http://39.96.12.43:8000/alipay/return/?charset=utf-8&out_trade_no=201702022ssswq&method=alipay.trade.page.pay.return&total_amount=100.00&sign=X59PxIrSunlz4IqhjBbHb1dzoPCChJbRN720AMn74kyDT54wkSCt1XIK%2B55PiWWU%2F3ytN7yn6Imt%2BaODtjPRF15VCCYrxDdby8tnBCxI3A7owvU9urvH5CwbHuvxE047ceHmVx44lA5%2BE9RUtUwd%2B%2Ba0omTdi83%2Fc6CPWlKLJPq1d5HIbJg8HEnPGqHObJadOkmpEpDcVRQyy74TibmAau9ZfHc4u4%2BuWyIzPJpnsEqjlhsaYraJdIUIEuG5qnO4dpNS66wyX%2FP7Ri3eb1JufklIl3CX9718XYf6IxQ5vrTjmw0PzieAH6ZvavYm2o%2BrH4H01D71mtVzQVJUc3Bqqg%3D%3D&trade_no=2019021722001471690500769031&auth_app_id=2016092400589241&version=1.0&app_id=2016092400589241&sign_type=RSA2&seller_id=2088102177131773&timestamp=2019-02-17+12%3A34%3A09


https://openapi.alipaydev.com/gateway.do?app_id=&biz_content=%7B%22subject%22%3A%2220190217220809366%22%2C%22out_trade_no%22%3A%2220190217220809366%22%2C%22total_amount%22%3A200.0%2C%22product_code%22%3A%22FAST_INSTANT_TRADE_PAY%22%7D&charset=utf-8&method=alipay.trade.page.pay&notify_url=http%3A%2F%2F39.96.12.43%3A8000%2Falipay%2Freturn%2F&return_url=http%3A%2F%2F39.96.12.43%3A8000%2Falipay%2Freturn%2F&sign_type=RSA2&timestamp=2019-02-17+22%3A26%3A09&version=1.0&sign=UuoIjRxBAdVb67zQzLPmZxO3OrDkJvlsh0UCw3OzLDre2HrIrNJaPCbWvtqC7RRvVwMVKnAaCeXGSU9HTHEvnP9Bi4oEevpeMZIcOtMBO9EMKbB1nzk84ROtP4rC6lEqZOeyvOJsi4NHFEh4bO8OXVvK2%2Fb34J9YtplSo5Ucl6qJDHTREjWf3mJ%2FwU4knougrSR269ze0dTGr%2FkjppldVJWjKtbcI5MOy1BrymInhw6qNq9zWQxAS7xpdNpUEWA0DESTK%2B5KQVzRvavGlgqHeFCUCkGH8UoVhMx2wRiLF9ylX4QpZ85kfbQb3uZ%2FmpBHjOjlunEjy5Vj3xypfGGlqg%3D%3D