# 视图集ViewSet

使用视图集ViewSet，可以将一系列逻辑相关的动作放到一个类中：

* list()  提供一组数据
* retrieve()  提供单个数据
* create()  创建数据
* update()  保存数据
* destory()  删除数据

ViewSet视图集类不再实现get()、post()等方法，而是实现动作 **action** 如 list() 、create() 等。

视图集只在使用as_view()方法的时候，才会将**action**动作与具体请求方式对应上。如：

```python
class BookInfoViewSet(viewsets.ViewSet):

    def list(self, request):
        ...

    def retrieve(self, request, pk=None):
        ...
```

在设置路由时，我们可以如下操作

```python
urlpatterns = [
    url(r'^books/$', BookInfoViewSet.as_view({'get':'list'}),
    url(r'^books/(?P<pk>\d+)/$', BookInfoViewSet.as_view({'get': 'retrieve'})
]
```

### action属性

在视图集中，我们可以通过action对象属性来获取当前请求视图集时的action动作是哪个。

例如：

```python
def get_serializer_class(self):
    if self.action == 'create':
        return OrderCommitSerializer
    else:
        return OrderDataSerializer
```

## 常用视图集父类

#### 1） ViewSet

继承自`APIView`，作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等。

在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法。

#### 2）GenericViewSet

继承自`GenericAPIView`，作用也与GenericAPIVIew类似，提供了get_object、get_queryset等方法便于列表视图与详情信息视图的开发。

#### 3）ModelViewSet

继承自`GenericAPIVIew`，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。

#### 4）ReadOnlyModelViewSet

继承自`GenericAPIVIew`，同时包括了ListModelMixin、RetrieveModelMixin。

## 视图集中定义附加action动作

在视图集中，除了上述默认的方法动作外，还可以添加自定义动作。

添加自定义动作需要使用`rest_framework.decorators.action`装饰器。

以action装饰器装饰的方法名会作为action动作名，与list、retrieve等同。

action装饰器可以接收两个参数：

* **methods**:  该action支持的请求方式，列表传递
* **detail**:  表示是action中要处理的是否是视图资源的对象（即是否通过url路径获取主键）
  * True 表示使用通过URL获取的主键对应的数据对象
  * False 表示不使用URL获取主键

举例：

```python
from rest_framework import mixins
from rest_framework.viewsets import GenericViewSet
from rest_framework.decorators import action

class BookInfoViewSet(mixins.ListModelMixin, mixins.RetrieveModelMixin, GenericViewSet):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    # detail为False 表示不需要处理具体的BookInfo对象
    @action(methods=['get'], detail=False)
    def latest(self, request):
        """
        返回最新的图书信息
        """
        book = BookInfo.objects.latest('id')
        serializer = self.get_serializer(book)
        return Response(serializer.data)

    # detail为True，表示要处理具体与pk主键对应的BookInfo对象
    @action(methods=['put'], detail=True)
    def read(self, request, pk):
        """
        修改图书的阅读量数据
        """
        book = self.get_object()
        book.bread = request.data.get('read')
        book.save()
        serializer = self.get_serializer(book)
        return Response(serializer.data)
```

url的定义

```python
urlpatterns = [
    url(r'^books/$', views.BookInfoViewSet.as_view({'get': 'list'})),
    url(r'^books/latest/$', views.BookInfoViewSet.as_view({'get': 'latest'})),
    url(r'^books/(?P<pk>\d+)/$', views.BookInfoViewSet.as_view({'get': 'retrieve'})),
    url(r'^books/(?P<pk>\d+)/read/$', views.BookInfoViewSet.as_view({'put': 'read'})),
]
```



## 视图集的继承关系

![视图集的继承关系](/images/视图集类继承关系.png)