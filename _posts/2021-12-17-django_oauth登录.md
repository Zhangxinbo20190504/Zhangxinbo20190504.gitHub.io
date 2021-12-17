---
layout:     post
title:      "oAuth登录"
subtitle:   "Django + Ant Design Vue实现oauth登录"
date:       2021-12-17 16:00:00
author:     "Xinbo"
header-img: "img/post-bg-universe.jpg"
tags:
    - Django
---


当前，有很多大型网站或者系统支持oauth登录，通过oauth登录，可以向这些大型网站请求用户信息，不用用户额外注册，减轻用户账号管理的负担，我们也将使用oauth的方式打造用户登录模块

## oAuth登录流程

oauth登录流程图如下：

![图片名称](https://s2.loli.net/2021/12/17/bBSewkxCy2jqhf9.png)

第一步: 网页后端发现用户未登录，请求身份验证
第二步: 用户登录后，开放平台生成登录预授权码，302跳转至重定向地址；
第三步: 网页后端调用获取登录用户身份校验登录预授权码合法性，获取到用户身份；
第四步: 如需其他用户信息，网页后端可调用[获取用户信息

## oAuth登录实现

> 第三方本次选择飞书，你也可以选择 微信 微博 github gitlab等，过程都差不多
>
> 登录实现将采用 django + ant design vue实现，环境搭建及前后端整合请参考另一篇博客[Django + Ant Design Vue 搭建web平台](https://zhangxinbo20190504.github.io/2021/11/23/Django-+-Ant-Design-Vue-%E6%90%AD%E5%BB%BAweb%E5%B9%B3%E5%8F%B0/)

### 前端实现

首先准备两个页面 A.vue 和 B.vue，并且在vue router的配置文件中配置相应的url: url_A url_B

A的作用主是呈现给用户的登录界面，在用户触发登录按钮时，向第三方网站发送请求，请求中带 redirect_uri 参数，用于第三方网站返回授权code重定向，redirect_uri 的值应该为 url_B

在A 中首先定义一个button

```vue
<a-button
          size="large"
          type="primary"
          @click="handleLoginClick()"
          class="login-button"
          :loading="state.loginBtn"
          :disabled="state.loginBtn"
        >
</a-button>
```

用户点击后触发 handleLoginClick() 方法

> handleLoginClick() 就是去请求第三方网站
>
> 请求url的格式根据第三方网站的不同可能略有区别，如有些还需要 app_secret这个参数，app_id 和 app_secret 在第三方网站注册应用的时候会给出
>
> 这里的url_B可能需要[URLEncode](https://meyerweb.com/eric/tools/dencoder/)处理

```js
...
methods: {
    handleLoginClick () {
      this.state.loginBtn = true
      console.log('Click')
      // 请求飞书预授权码
      const url = 'https://open.feishu.cn/open-apis/authen/v1/index?app_id=<your app id>&redirect_uri=<url_B>'
      location.href = url
      console.log('Finish Jump')
    }
}
...
```

A的作用也就完成了

B作为重定向的页面，负责接收第三方网站的授权code，并且用这个code去请求后台数据

在 created 中接收到code

```js
created () {
    var _this = this
    console.log('Redirect Here')
    console.log('code==>', this.$route.query.code)
    _this.code = this.$route.query.code
    _this.autoLogin()
  },
```

autoLogin中用这个code去请求后台，调用vuex中的Login去请求后台的url，并定义成功和失败时的具体动作（这里省略）

```js
<script>
import { mapActions } from 'vuex'
export default {
	...
	methods: {
    	...mapActions(['Login', 'Logout']),
        autoLogin () {
          const {
            state,
            Login
          } = this
          const loginParams = {
            code: this.code
          }
          Login(loginParams)
            .then((res) => this.loginSuccess(res))
            .catch(err => this.requestFailed(err))
            .finally(() => {
              state.loginBtn = false
          })
        },
	...
}
</script>
```

到此前端实现结束

### 后台实现

后台接收到前端传过来的code，执行登录操作

定义一个authenticate app，并加入到 django 的 settings.py 中

```shell
python3 manage.py startapp authenticate
```

```python
INSTALLED_APPS = [
    ...
    'authenticate',
    ...
]
```

 在 authenticate/views.py 中定义登录视图

> 这里失败的返回码自定义，可在前端校验返回码，根据返回码不同显示不同的用户反馈

```python
@require_http_methods(["POST"])
def login_view(request):
    code = request.POST["code"]
    credentials = {'code': code}
    user = authenticate(request=None, **credentials)
    if user:
        login(request, user)
        # do your extra options here
        return JsonResponse({"code": 200, "message": ""})
    else:
        return JsonResponse({"code": 205, "message": ""})
```

并在 authenticate/urls.py 中定义视图的url

```python
from django.conf.urls import url
from .views import login_view
app_name = 'authenticate'
urlpatterns = [
	...
    url(r'^auth/login$', login_view, name='login'),
    ...
]
```

并在 mysite/urls.py （Django根url，视自己情况而定）中引入 authenticate app 的 url

```python
urlpatterns = [
    ...
    url(r'^api/', include('authenticate.urls')),  # 认证url
    ...
]
```

####  authenticate方法原理

我们的认证”证书“ credentials = {'code': code} ，并不是传统意义上的用户名密码，首先来看一下django的 authenticate 实现

```python
def authenticate(request=None, **credentials):
    """
    If the given credentials are valid, return a User object.
    """
    for backend, backend_path in _get_backends(return_tuples=True):
        try:
            inspect.getcallargs(backend.authenticate, request=request, **credentials)
        except TypeError:
            # This backend doesn't accept these credentials as arguments. Try the next one.
            continue
        try:
            user = backend.authenticate(request=request, **credentials)
        except PermissionDenied:
            # This backend says to stop in our tracks - this user should not be allowed in at all.
            break
        # Annotate the user object with the path of the backend.
        if user is None:
            continue
        user.backend = backend_path
        return user
```

首先 for 循环会遍历django注册的认证后端

inspect.getcallargs(backend.authenticate, request=request, **credentials)

然后通过这句话会找到我们的credentials是否跟认证后端的 authenticate 方法参数匹配，如果匹配则选择对应的认证后端backend，并且调用backend的authenticate方法，返回一个user对象

显然，对于我们这种类型的 credentials = {'code': code}  证书，是没有认证后端可以帮我们认证的，所以必须自己实现认证后端

#### 自定义认证后端

在 authenticate 目录下 新建 backend.py

```python
# *飞书后端认证
class LarkBackend(object):
	...
    #
    # The Django auth backend API
    #
    def authenticate(self, code, **kwargs):
        try:
            print('Code==>', code)
            self._get_app_access_token()
            self._get_lark_user(code)
            return self._get_or_create_user()
        except Exception:
            print(traceback.format_exc())
            return
    
    def get_user(self, user_id):
        user = None
        try:
            user = User.objects.get(pk=user_id)
        except Exception:
            pass
        return user
```

> get_user 方法在logout中会使用

这个认证后端必须实现两个方法 authenticate get_user

> authenticate 认证的具体细节视个人业务情况而定，这里不做展示

authenticate 如果成功则返回一个Django user对象

在 django settings.py中注册我们的自定义后端

```python
AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'authenticate.backend.LarkBackend',  # 配置为使用飞书认证
)
```

django执行authenticate 方法时会按照注册顺序依次调用后端的authenticate方法，直到返回一个user对象



至此，实现oauth登录