---
title: Django-auth模块源码解析
date: 10-04-2022
categories: 
- python
tags: 
- django
---


## Django-Auth 模块解析

> 源自何处？
> 1. 需要在django的admin、rest framework模块和JWT中,用三方的cas账号登录
> 2. 需要把三方账号提供的用户信息存进 auth_user的表中
> 3. 总的来说就是:用三方账号登录django,并和django的user表对接上

### 1. 首先感性认识一下auth

1. auth就像django项目中一个独立的app, 其中有models.py admin.py views.py 等
2. auth的路径: django\contrib\auth
3. 用django命令创建一个项目时,settings.INSTALLED_APPS中已经默认加入了auth这个模块
```python
INSTALLED_APPS = [
    ...
    'django.contrib.auth',
    ...
]
```
4. 因此, 创建django项目后,进行 python manage.py migrate 同步数据库时,会自动将auth app的models同步到数据库
5. auth/models.py 中定义了 User(用户表),Group(用户组表),Permission(权限表),AnonymousUser(匿名用户类)
6. auth/\_\_init\_\_.py中,定义了许多用户认证相关的方法:authenticate,login,logout,get_user等等
7. 一个用户可以关联多个权限,一个组也可以关联多个权限
8. 一个用户拥有的所有权限指:用户关联的所有权限 + 所在组关联的所有权限

### 2. 看一下auth app中有哪些表

> django\contrib\auth\models.py

#### 2.1 auth_user(用户),其中每个字段都是用户基本信息

1. is_superuser: 是否是超级用户
2. is_staff:是否可以登录后台管理系统
3. is_active:False表示用户未激活,可以理解为被虚拟删除了
```mysql
+--------------+--------------+------+-----+---------+----------------+
| Field        | Type         | Null | Key | Default | Extra          |
+--------------+--------------+------+-----+---------+----------------+
| id           | int(11)      | NO   | PRI | NULL    | auto_increment |
| password     | varchar(128) | NO   |     | NULL    |                |
| last_login   | datetime(6)  | YES  |     | NULL    |                |
| is_superuser | tinyint(1)   | NO   |     | NULL    |                |
| username     | varchar(150) | NO   | UNI | NULL    |                |
| first_name   | varchar(150) | NO   |     | NULL    |                |
| last_name    | varchar(150) | NO   |     | NULL    |                |
| email        | varchar(254) | NO   |     | NULL    |                |
| is_staff     | tinyint(1)   | NO   |     | NULL    |                |
| is_active    | tinyint(1)   | NO   |     | NULL    |                |
| date_joined  | datetime(6)  | NO   |     | NULL    |                |
+--------------+--------------+------+-----+---------+----------------+
11 rows in set (0.01 sec)
```
#### 2.2 auth_group(用户组),一条数据表示一个用户组

```mysql
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int(11)      | NO   | PRI | NULL    | auto_increment |
| name  | varchar(150) | NO   | UNI | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
2 rows in set (0.01 sec)
```

#### 2.3 auth_user_groups(用户与组的关联关系)

1. 一个用户可以属于多个组
2. 一个组可以拥有多个用户
3. 用户和组是多对多的关系

```mysql
+----------+---------+------+-----+---------+----------------+
| Field    | Type    | Null | Key | Default | Extra          |
+----------+---------+------+-----+---------+----------------+
| id       | int(11) | NO   | PRI | NULL    | auto_increment |
| user_id  | int(11) | NO   | MUL | NULL    |                |
| group_id | int(11) | NO   | MUL | NULL    |                |
+----------+---------+------+-----+---------+----------------+
3 rows in set (0.00 sec)
```
4. 看一下models.py中对两者关联关系的定义
- User模型和group是多对多关系
- User模型和permission(用户权限,后面会提到)也是多对多关系

```python

class User(AbstractUser):
    """省略"""
class AbstractUser(AbstractBaseUser, PermissionsMixin):
    """省略"""

class PermissionsMixin(models.Model):
    """
    Add the fields and methods necessary to support the Group and Permission
    models using the ModelBackend.
    """
    is_superuser = models.BooleanField(
        _('superuser status'),
        default=False,
        help_text=_(
            'Designates that this user has all permissions without '
            'explicitly assigning them.'
        ),
    )
    # 以下关系体现在 auth_user_groups 表中
    groups = models.ManyToManyField(
        Group,
        verbose_name=_('groups'),
        blank=True,
        help_text=_(
            'The groups this user belongs to. A user will get all permissions '
            'granted to each of their groups.'
        ),
        related_name="user_set",
        related_query_name="user",
    )
    # 以下关系体现在 auth_user_user_permissions 表中
    user_permissions = models.ManyToManyField(
        Permission,
        verbose_name=_('user permissions'),
        blank=True,
        help_text=_('Specific permissions for this user.'),
        related_name="user_set",
        related_query_name="user",
    )
    "以下省略"
```

### 3. User模型提供的一些方法

1. 如以下代码: User model中定义了一个UserManager的实例objects,UserManager中定义了一些方法,例如创建用户等等

```python
class AbstractUser(AbstractBaseUser, PermissionsMixin):
    """...省略"""
    objects = UserManager()
    """...省略"""

class UserManager(BaseUserManager):
    use_in_migrations = True
    """...省略其余方法"""
    def create_user(self, username, email=None, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', False)
        extra_fields.setdefault('is_superuser', False)
        return self._create_user(username, email, password, **extra_fields)

    def create_superuser(self, username, email=None, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_staff') is not True:
            raise ValueError('Superuser must have is_staff=True.')
        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self._create_user(username, email, password, **extra_fields)
    """...省略其余方法"""
```
2. 因此可以通过调用User.objects.create_user()创建用户,或者其他实用方法对用户进行各种操作

```python
from django.contrib.auth.models import User
user = User.objects.create_user(username="username", password="password")
```

### 4. Django的admin是如何进行用户身份认证的?（注意只是认证身份的合法性，并没有记录“是否登录”，后面详述）

1. 在django.contrib.auth.\_\_init\_\_.py文件中,定义了许多用户认证相关的方法
2. 包括 login, logout, authenticate, get_user, get_user_model等等
3. 其中authenticate是django中用来验证用户是否合法的关键函数,它接收,request,credentials等参数,如果验证合法,则返回一个user对象,否则返回None
4. authenticate方法的执行流程分析如下:
    - 调用 **_get_backends** 方法,获取backends列表
    - 遍历backends列表,调用每一个backend的authenticate方法,只要有一个backend的authenticate方法返回user对象,表示认证通过
    - 因此可以理解为:backends中存放了多种用户认证的方式,用户只要通过了其中一个认证方式就认为认证通过
    - **扩展:可以通过自定义多个不同的backends,加入backends列表,实现各种三方身份认证方式**
    - **注意: 自定义的backend,一定要确保backend.authenticate方法返回一个合法的user对象**
    - 后面会提到如何自定义backend类

```python
# django\contrib\auth\__init__.py::authenticate方法
@sensitive_variables('credentials')
def authenticate(request=None, **credentials):
    """
    If the given credentials are valid, return a User object.
    """
    # 遍历认证backends，有一个返回user，任务认证通过，并返回user
    for backend, backend_path in _get_backends(return_tuples=True):
        backend_signature = inspect.signature(backend.authenticate)
        try:
            backend_signature.bind(request, **credentials)
        except TypeError:
            # This backend doesn't accept these credentials as arguments. Try the next one.
            continue
        try:
            user = backend.authenticate(request, **credentials)
        except PermissionDenied:
            # This backend says to stop in our tracks - this user should not be allowed in at all.
            break
        if user is None:
            continue
        # Annotate the user object with the path of the backend.
        user.backend = backend_path
        return user

    # The credentials supplied are invalid to all backends, fire signal
    user_login_failed.send(sender=__name__, credentials=_clean_credentials(credentials), request=request)
```

5. 如何获取认证用的backends呢?_get_backends方法干了什么?
    - _get_backends方法实际读取了settings.AUTHENTICATION_BACKENDS  
  
```python
def _get_backends(return_tuples=False):
    backends = []
    # 读取backends列表 settings.AUTHENTICATION_BACKENDS
    for backend_path in settings.AUTHENTICATION_BACKENDS:
        backend = load_backend(backend_path)
        backends.append((backend, backend_path) if return_tuples else backend)
    if not backends:
        raise ImproperlyConfigured(
            'No authentication backends have been defined. Does '
            'AUTHENTICATION_BACKENDS contain anything?'
        )
    return backends
```

6. settings.AUTHENTICATION_BACKENDS是什么?
- AUTHENTICATION_BACKENDS是定义在django settings中的一个配置项
- django项目生成的settings.py文件中未找到AUTHENTICATION_BACKENDS字段
- django中还有一部分配置写在**django\conf\global_settings.py**文件中
- 可以看到,django默认只有一个backend:**django.contrib.auth.backends.ModelBackend**
- 因此可以理解为: django/contrib/auth/\_\_init\_\_.py 中的authenticate 只使用了django.contrib.auth.backends.ModelBackend这一个backend,进行了用户认证;
- 我们可以自定义一个backend,在settings.py中覆盖AUTHENTICATION_BACKENDS的配置,将默认ModelBackend和自定义的backend都放进去
- 让django在通过ModelBackend验证失败时,能够通过自定义backend的authenticate验证通过

```python
# django\conf\global_settings.py
AUTHENTICATION_BACKENDS = ['django.contrib.auth.backends.ModelBackend']
```

7. django.contrib.auth.backends.ModelBackend是如何进行用户身份认证的?
    - 下方代码只展示了ModelBackend类中authenticate和user_can_authenticate两个方法,其他方法可翻阅源码查看
    - authenticate方法通过"用户名"和"密码",验证用户是否合法. 不合法时,返回None;合法时,返回user对象
    - ModelBackend还提供了get_user_permissions, get_group_permissions,get_group_permissions等多个方法

```python
# django\contrib\auth\backends.py
class ModelBackend(BaseBackend):
    """
    Authenticates against settings.AUTH_USER_MODEL.
    """

    def authenticate(self, request, username=None, password=None, **kwargs):
        if username is None:
            username = kwargs.get(UserModel.USERNAME_FIELD)
        if username is None or password is None:
            return
        try:
            user = UserModel._default_manager.get_by_natural_key(username)
        except UserModel.DoesNotExist:
            # Run the default password hasher once to reduce the timing
            # difference between an existing and a nonexistent user (#20760).
            UserModel().set_password(password)
        else:
            if user.check_password(password) and self.user_can_authenticate(user):
                return user

    def user_can_authenticate(self, user):
        """
        Reject users with is_active=False. Custom user models that don't have
        that attribute are allowed.
        """
        is_active = getattr(user, 'is_active', None)
        return is_active or is_active is None

```

8. ModelBackend中的那么多方法如何调用呢?
- 看回 django\contrib\auth\__init__.py 中的authenticate方法
- 可以看到在backend.authenticate返回user对象后,执行了"user.backend = backend_path"代码
- 而经过authenticate返回的user,最终会被挂在request对象上,传给前端请求的各个view方法中

```python
# django\contrib\auth\__init__.py
@sensitive_variables('credentials')
def authenticate(request=None, **credentials):
    """
    If the given credentials are valid, return a User object.
    """
    for backend, backend_path in _get_backends(return_tuples=True):
        backend_signature = inspect.signature(backend.authenticate)
        try:
            backend_signature.bind(request, **credentials)
        except TypeError:
            # This backend doesn't accept these credentials as arguments. Try the next one.
            continue
        try:
            user = backend.authenticate(request, **credentials)
        except PermissionDenied:
            # This backend says to stop in our tracks - this user should not be allowed in at all.
            break
        if user is None:
            continue
        # Annotate the user object with the path of the backend.
        user.backend = backend_path
        return user

    # The credentials supplied are invalid to all backends, fire signal
    user_login_failed.send(sender=__name__, credentials=_clean_credentials(credentials), request=request)
```

- 因此在各个app的views中,我们可以通过以下方式调用backend中的例如get_user_permissions方法

```python
# 某app/views.py
from django.http import HttpResponse
def demo_view(request):
  perms = request.user.backend.get_user_permissions()
  return HttpResponse(perms)
```

### 5. 如何自定义一个Backend,通过邮箱和密码进行身份认证?

#### 5.1 创建一个EmailBackend,继承BaseBackend或ModelBackend,实现authenticate方法

1. 继承ModelBackend,可以继承ModelBackend自带的一些方法,例如get_user_permissions等等
2. 继承BaseBackend时,就没有get_user_permissions等等这些方法可用了

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend

UserModel = get_user_model()

class EmailBackend(ModelBackend):
  def authenticate(self, request, username=None, password=None, **kwargs):
    # 1. 获取邮箱
    email = kwargs.get(UserModel.EMAIL_FIELD, kwargs.get(UserModel.USERNAME_FIELD, username))
    # 2. 未填写邮箱,返回None,认证不通过
    if email is None or password is None:
        return
    try:
        # 3. 通过邮箱获取用户实例
        user = UserModel.objects.get(email=email)
    except UserModel.DoesNotExist:
        # Run the default password hasher once to reduce the timing
        # difference between an existing and a nonexistent user (#20760).
        UserModel().set_password(password)
    else:
      # 4. 密码正确 且 用户活跃状态 返回user实例,表示认证通过
        if user.check_password(password) and self.user_can_authenticate(user):
            return user
```

#### 5.2 在settings.py中配置AUTHENTICATION_BACKENDS参数

1. 在settings.py文件中将自定义的EmailBackend放进AUTHENTICATION_BACKENDS中
2. 注意,django默认的django.contrib.auth.backends.ModelBackend也要放进去

```python
# settings.py
# 假设EmailBackend相对于项目根目录的路径是 "auth.backends.py"
AUTHENTICATION_BACKENDS = ['django.contrib.auth.backends.ModelBackend', 'auth.backends.EmailBackend',]
```
#### 5.3 重启django项目,即可通过"邮箱"+"密码"的方式登录django后台

#### 5.4 用三方账号认证时，该怎么做？
1. 自定义一个Backend，继承BaseBackend或ModelBackend,实现authenticate方法
2. 在authenticate方法中，验证三方账号的用户信息，将用户信息保存或更新在User中（实现三方用户与User表的对接）
3. 在authenticate中返回一个user对象
4. 在settings.py中配置AUTHENTICATION_BACKENDS参数，新增自定义的Backend

### 6. ModelBackend.authenticate中的UserModel是什么?为什么不是auth.models.User?

1. 首先看下django/contrib/auth/backends.py:ModelBackend
2. UserModel是**get_user_model**方法的返回值,看起来UserModel应该指代的是一个User Model

```python
# 首先看下django/contrib/auth/backends.py
UserModel = get_user_model()
from django.contrib.auth import get_user_model
class ModelBackend(BaseBackend):
    def authenticate(self, request, username=None, password=None, **kwargs):
        '''...省略'''
        try:
            user = UserModel._default_manager.get_by_natural_key(username)
        except UserModel.DoesNotExist:
            # Run the default password hasher once to reduce the timing
            # difference between an existing and a nonexistent user (#20760).
            UserModel().set_password(password)
        '''...省略'''
```

3. 看下**get_user_model**方法,它根据**settings.AUTH_USER_MODEl**,返回一个用户模型

```python
# django.contrib.auth.get_user_model
def get_user_model():
    """
    Return the User model that is active in this project.
    """
    try:
        # 根据settings中的AUTH_USER_MODEL,获取指定的 用户模型
        return django_apps.get_model(settings.AUTH_USER_MODEL, require_ready=False)
    except ValueError:
        raise ImproperlyConfigured("AUTH_USER_MODEL must be of the form 'app_label.model_name'")
    except LookupError:
        raise ImproperlyConfigured(
            "AUTH_USER_MODEL refers to model '%s' that has not been installed" % settings.AUTH_USER_MODEL
        )
```
4. settings.AUTH_USER_MODEl是什么?看下**global_settings.py**,看到它指的是auth.User

```python
# django/conf/global_settings.py:508
AUTH_USER_MODEL = 'auth.User'
```

5. 以上说明,Django默认的身份认证Backend,根据settings.AUTH_USER_MODEL使用指定的用户模型,默认为django.contrib.auth.models.py下的User模型.

6. 因此,如果我们想改造用户模型,比如基于auth.User扩展字段,那么我们可以新建一个自己的用户模型MyUser,之后在settings.py中覆盖AUTH_USER_MODEL字段

7. 自定义用户模型时,需要在用户模型中实现一些额外的方法. 这些方法是指: UserModel通过点"."操作符调用用户模型的那些自有方法.

### 7. django的rest-framework(简称DRF)模块只能通过用户名密码和sessions进行身份认证?

#### 1. DRF的身份认证流程
1. settings.py设置DRF的认证类
    - 一般在使用了DRF的django项目中,都要在settings.py中对DRF进行配置
    - 比如"DEFAULT_AUTHENTICATION_CLASSES"字段,是用来设置DRF的认证类的
    - 默认包含DRF自带的两个认证类: BasicAuthentication SessionAuthentication

    ```python
    REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',

    ),
    ```
2. 默认的DRF认证类--rest_framework.authentication.BasicAuthentication
    - authenticate首先获取request.headers的auth
    - 在auth不满足一些条件(长度为2)时,返回None,认证失败
    - 对auth进行解析,从auth中提取出userid和password
    - 将用户名密码传给authenticate_credentials方法,并返回authenticate_credentials的返回值

    - authenticate_credentials方法调用django.contrib.auth.authenticate方法,并返回.
    - django.contrib.auth.authenticate就是**文章第4部分**开始提到的django自带的用户身份认证的方法,它通过遍历backends,验证认证是否通过

    - 以上概括起来:DRF默认的身份认证流程是:1)通过request.headers读取用户id和密码; 2)调用django自带的身份认证方法django.contrib.auth.authenticate; 3)认证通过,返回user,不通过则返回None

```python
# rest_framework/authentication.py::BasicAuthentication
class BasicAuthentication(BaseAuthentication):
    """
    HTTP Basic authentication against username/password.
    """
    www_authenticate_realm = 'api'

    def authenticate(self, request):
        """
        Returns a `User` if a correct username and password have been supplied
        using HTTP Basic authentication.  Otherwise returns `None`.
        """
        auth = get_authorization_header(request).split()

        if not auth or auth[0].lower() != b'basic':
            return None

        if len(auth) == 1:
            msg = _('Invalid basic header. No credentials provided.')
            raise exceptions.AuthenticationFailed(msg)
        elif len(auth) > 2:
            msg = _('Invalid basic header. Credentials string should not contain spaces.')
            raise exceptions.AuthenticationFailed(msg)

        try:
            try:
                auth_decoded = base64.b64decode(auth[1]).decode('utf-8')
            except UnicodeDecodeError:
                auth_decoded = base64.b64decode(auth[1]).decode('latin-1')
            auth_parts = auth_decoded.partition(':')
        except (TypeError, UnicodeDecodeError, binascii.Error):
            msg = _('Invalid basic header. Credentials not correctly base64 encoded.')
            raise exceptions.AuthenticationFailed(msg)

        userid, password = auth_parts[0], auth_parts[2]
        return self.authenticate_credentials(userid, password, request)

    def authenticate_credentials(self, userid, password, request=None):
        """
        Authenticate the userid and password against username and password
        with optional request for context.
        """
        credentials = {
            get_user_model().USERNAME_FIELD: userid,
            'password': password
        }
        user = authenticate(request=request, **credentials)

        if user is None:
            raise exceptions.AuthenticationFailed(_('Invalid username/password.'))

        if not user.is_active:
            raise exceptions.AuthenticationFailed(_('User inactive or deleted.'))

        return (user, None)

    def authenticate_header(self, request):
        return 'Basic realm="%s"' % self.www_authenticate_realm

```

3. 默认的DRF认证类--rest_framework.authentication.SessionAuthentication
   - 这个类通过session会话判断是否认证通过，直接读取request._request.user,user存在表示认证通过。
   - _request是原始request
   - 之所以可以直接获取request._request.user，是因为：django的request到SessionAuthentication之前，已经经过了django的SessionMiddleware了，这个中间件会进行session的验证。

4. BasicAuthentication只能通过用户名密码认证的原因
  - 从上一步分析中,可知:默认的DRF认证流程,必须通过"userid和password"的提取,一旦这一步不通过,那么整个认证就不通过.
  - 而我们需要扩展的认证方式是没有用户名密码的,所以无法通过drf的认证.


#### 2. 如何在DRF中,通过其他方式(例如.用CAS的headers信息，而非用户名密码)进行身份认证?
###### 方法1. 模仿BasicAuthentication,实现一个自己的例如 CasRestAuthentication类,绕过"userid和password"的提取
  - 新的CasRestAuthentication类,继承BaseAuthentication
  - 覆盖BasicAuthentication的authenticate和authenticate_header方法，authenticate_header方法是用来返回“认证失败时附加在headers上的值”，可以不实现
  - 新的authenticate不再读取userid和password,而是直接调用django自带的django.contrib.auth.authenticate，传入CasBackend认证需要的参数cas_headers,进行认证
  - django.contrib.auth.authenticate是通过多个Backends遍历认证的,只要我们实现了CasBackend,并配置在settings.AUTHENTICATION_BACKENDS,DRF就可以通过认证.

```python
# auth/rest_authentication.py
from django.contrib.auth import authenticate
from django.utils.translation import gettext_lazy as _

from rest_framework.authentication import BaseAuthentication
from rest_framework import exceptions


class CasRestAuthentication(BaseAuthentication):
    def authenticate(self, request):
        # 这里,直接调用authenticate方法,并传入CasBackend认证需要的认证参数，就可以让DRF直接走django自带的authenticate方法进行身份认证了
        user = authenticate(request, cas_headers=request.headers)
        if user is None:
            raise exceptions.AuthenticationFailed(_('Invalid username/password.'))

        if not user.is_active:
            raise exceptions.AuthenticationFailed(_('User inactive or deleted.'))
        return (user, None)

    def authenticate_header(self, request):
      return ""
```

  - 将CasRestAuthentication配置在settings.py中

```python
# settings.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        # 以下为自定义的CasRestAuthentication
        'auth.rest_authentication.CasRestAuthentication',
    ),

}
```

###### 方法2. 用djangorestframework-simplejwt实现基于token的DRF认证方式
1. [djangorestframework-simplejwt的使用方法](https://www.jianshu.com/p/7ebf659c57a3)
2. simplejwt的大概功能
   - simplejwt提供了两个view：TokenObtainPairView和TokenRefreshView
   - **TokenObtainPairView**的作用：根据用户名密码，生成两个token：accessToken、refreshToken
   - 此后，在其他请求的headers中带上accessToken，DRF就认为该用户认证通过
   - accessToken是有有效期的，accessToken过期后，用refreshToken请求**TokenRefreshView**生成一个新的accessToken，直到更新后的accessToken也过期了

```python
# urls.py
urlpatterns = [
  url(r'api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
  url(r'api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
]
```

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # 加上JWT的认证方式，JWTAuthentication能够通过accessToken对用户的身份进行认证
        'rest_framework_simplejwt.authentication.JWTAuthentication',

        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ),

}
```

3. 重新实现TokenObtainPairView，使得通过“Cas headers”或“用户名密码”都能够拿到token对

```python
# auth/views.py 项目中自定义的auth模块
# 继承TokenObtainPairView
from rest_framework_simplejwt.views import TokenObtainPairView
class CasTokenObtainPairView(TokenObtainPairView):
    # 自定义一个 CasTokenObtainPairSerializer，该序列化器对传入的认证参数进行validate验证
    serializer_class = CasTokenObtainPairSerializer

    def post(self, request, *args, **kwargs):
        data = request.data

        # 改动1：原来serializer只传request.data;为了实现cas的认证，此处补充传了cas的headers
        # cas_headers参数用于在序列化器中作为认证参数，通过验证
        data["cas_headers"] = request.headers

        serializer = self.get_serializer(data=data)

        try:
            serializer.is_valid(raise_exception=True)
        except TokenError as e:
            raise InvalidToken(e.args[0])

        # 改动2：调用auth.login,保存用户的session
        login(request=request, user=serializer.user)

        return Response(serializer.validated_data, status=status.HTTP_200_OK)
```

```python
# auth/serializer.py 项目中自定义的auth模块
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
class CasTokenObtainPairSerializer(TokenObtainPairSerializer):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 改写1：认证方式除了用户名密码以外，加入Cas的请求头
        self.fields['cas_headers'] = serializers.JSONField(allow_null=True, required=False)

        # 改写2：用户名密码可以为空
        self.fields[self.username_field] = serializers.CharField(allow_blank=True, required=False)
        self.fields['password'] = PasswordField(allow_blank=True, required=False)

    def validate(self, attrs):
        # 改写3：认证参数加入Cas请求头，供CasBackend使用
        authenticate_kwargs = {
            'cas_headers': attrs['cas_headers'],
        }
        if attrs.get(self.username_field):
            authenticate_kwargs[self.username_field] = attrs.get(self.username_field)
        if attrs.get('password'):
            authenticate_kwargs['password'] = attrs.get('password')

        # 1. 调用django.contrib.auth.authenticate，进行用户的身份认证
        self.user = authenticate(**authenticate_kwargs)

        # 2. 认证不通过
        if not api_settings.USER_AUTHENTICATION_RULE(self.user):
            raise exceptions.AuthenticationFailed(
                self.error_messages['no_active_account'],
                'no_active_account',
            )

        # 3. 认证通过，准备生成token
        data = {}

        refresh = self.get_token(self.user)

        data['refresh'] = str(refresh)
        data['access'] = str(refresh.access_token)

        if api_settings.UPDATE_LAST_LOGIN:
            update_last_login(None, self.user)

        # 4. 返回token
        return data
```


### 8. 登录与退出登录

#### 1. 身份认证和登录有什么区别？

1. 身份认证：仅判断用户身份是否合法，并返回一个合法的user对象；如果身份不合法，则返回None
2. 登录：在身份认证通过的前提下，为该用户生成一系列session信息，或cookies，或token

3. 身份认证一般被登录流程所使用，或者在中间件中使用
4. 登录一般只使用一次，即前端发起一次登录请求

5. 登录方式有3种：sessions、cookies、token

#### 2. django.contrib.auth.login
> sessions的登录方式，admin只认sessions
1. login方法接收一个user，user对象是一个经过认证的user对象
2. 之后为该用户生成一系列sessions信息，并保存在session数据库里
3. 在request的cookies里填充sessionid
4. 此后的请求只要cookies里携带sessonid，就表示已登录（相当于 记录用户的登录状态）

```python
def login(request, user, backend=None):
    """
    Persist a user id and a backend in the request. This way a user doesn't
    have to reauthenticate on every request. Note that data set during
    the anonymous session is retained when the user logs in.
    """
    session_auth_hash = ''
    if user is None:
        user = request.user
    if hasattr(user, 'get_session_auth_hash'):
        session_auth_hash = user.get_session_auth_hash()

    if SESSION_KEY in request.session:
        if _get_user_session_key(request) != user.pk or (
                session_auth_hash and
                not constant_time_compare(request.session.get(HASH_SESSION_KEY, ''), session_auth_hash)):
            # To avoid reusing another user's session, create a new, empty
            # session if the existing session corresponds to a different
            # authenticated user.
            request.session.flush()
    else:
        request.session.cycle_key()

    try:
        backend = backend or user.backend
    except AttributeError:
        backends = _get_backends(return_tuples=True)
        if len(backends) == 1:
            _, backend = backends[0]
        else:
            raise ValueError(
                'You have multiple authentication backends configured and '
                'therefore must provide the `backend` argument or set the '
                '`backend` attribute on the user.'
            )
    else:
        if not isinstance(backend, str):
            raise TypeError('backend must be a dotted import path string (got %r).' % backend)

    request.session[SESSION_KEY] = user._meta.pk.value_to_string(user)
    request.session[BACKEND_SESSION_KEY] = backend
    request.session[HASH_SESSION_KEY] = session_auth_hash
    if hasattr(request, 'user'):
        request.user = user
    rotate_token(request)
    user_logged_in.send(sender=user.__class__, request=request, user=user)
```

#### 3. django.contrib.auth.logout
1. django自带的auth.\_\_init\_\_.py中提供了一个函数logout
2. logout的作用是：清空请求的用户在django中保存的sessions，如果请求的用户为匿名用户，logout也会静默通过
3. 调用logout之后，客户端再次访问admin或者DRF的网页版，将处于匿名用户状态
4. logout函数与JWT的token无关，因此即使调用logout清空了sessions，通过未过期的accessToken，依然可以请求 DRF的api

#### 4. 梳理admin、DRF、JWT的认证登录流程
###### 1. admin（只认sessions）
1. 认证：通过django.contrib.auth.authenticate的认证
2. 登录：调用django.contrib.auth.login方法，实现sessions数据的生成

###### 2. DRF（网页版，只认sessions）
1. 认证：经过rest_framework.authentication.BasicAuthentication，用户名密码 认证
2. 登录：调用django.contrib.auth.login方法，实现sessions数据的生成

###### 3. DRF(JWT版)
1. 认证：方式一：获取token对，accessToken和refreshToken | 方式二：已生成的sessions
2. 登录：在accessToken有效期内，带着accessToken在headers里，就认为用户是登录态
3. 注意：token的方式未产生sessions或cookies，因此有了accessToken不能进入admin或DRF的网页版

###### 4. 总结
1. 前端页面通过JWT登录django后端后，跳转到admin页面并不能直接进入，因为JWT并没有生成sessions；
2. 如果要实现在通过JWT登录django后端时，将登录状态同步到DRF网页版、admin，可以在JWT的TokenObtainPairView视图函数中，调用auth.login方法，在生成token对的同时，生成sessions；















