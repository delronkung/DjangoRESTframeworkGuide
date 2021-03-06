# 视图说明

## 1.  两个基类

### 1）APIView

`rest_framework.views.APIView`

`APIView`是REST framework提供的所有视图的基类，继承自Django的`View`父类。

`APIView`与`View`的不同之处在于：

* 传入到视图方法中的是REST framework的`Request`对象，而不是Django的`HttpRequeset`对象；
* 视图方法可以返回REST framework的`Response`对象，视图会为响应数据设置（render）符合前端要求的格式；
* 任何`APIException`异常都会被捕获到，并且处理成合适的响应信息；
* 在进行dispatch()分发前，会对请求进行身份认证、权限检查、流量控制。

##### 支持定义的属性：

* **authentication_classes**   列表或元祖，身份认证类
* **permissoin_classes**   列表或元祖，权限检查类
* **throttle_classes**  列表或元祖，流量控制类

在`APIView`中仍以常规的类视图定义方法来实现get() 、post() 或者其他请求方式的方法。

举例：

```python
from rest_framework.views import APIView
from rest_framework.response import Response

# url(r'^books/$', views.BookListView.as_view()),
class BookListView(APIView):
    def get(self, request):
        books = BookInfo.objects.all()
        serializer = BookInfoSerializer(books, many=True)
        return Response(serializer.data)
```

### 2）GenericAPIView

`rest_framework.generics.GenericAPIView`

继承自`APIVIew`，增加了对于列表视图和详情视图可能用到的通用支持方法。通常使用时，可搭配一个或多个Mixin扩展类。

##### 支持定义的属性：

* 列表视图与详情视图通用：
  * **queryset**  列表视图的查询集
  * **serializer_class**  视图使用的序列化器
* 列表视图使用：
  * **pagination_class**  分页控制类
  * **filter_backends**  过滤控制后端
* 详情页视图使用：
  * **lookup_field**  查询单一数据库对象时使用的条件字段，默认为'`pk`'
  * **lookup_url_kwarg**  查询单一数据时URL中的参数关键字名称，默认与**look_field**相同    

##### 提供的方法：

* 列表视图与详情视图通用：

  * **get_queryset(self)** 

    返回视图使用的查询集，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，可以重写，例如：

    ```python
    def get_queryset(self):
        user = self.request.user
        return user.accounts.all()
    ```

  * **get_serializer_class(self)**   

    返回序列化器类，默认返回`serializer_class`，可以重写，例如：

    ```python
    def get_serializer_class(self):
        if self.request.user.is_staff:
            return FullAccountSerializer
        return BasicAccountSerializer
    ```

  * ##### get_serializer(self, *args, **kwargs)

    返回序列化器对象，被其他视图或扩展类使用，如果我们在视图中想要获取序列化器对象，可以直接调用此方法。

    **注意，在提供序列化器对象的时候，REST framework会向对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。**

* 详情视图使用：

  * **get_object(self)**
    返回详情视图所需的模型类数据对象，默认使用`lookup_field`参数来过滤queryset。
    在试图中可以调用该方法获取详情信息的模型类对象。

    **若详情访问的模型类对象不存在，会返回404。**

    **该方法会默认使用APIView提供的check_object_permissions方法检查当前对象是否有权限被访问。**

举例：

```python
# url(r'^books/(?P<pk>\d+)/$', views.BookDetailView.as_view()),
class BookDetailView(GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request, pk):
        book = self.get_object()
        serializer = self.get_serializer(book)
        return Response(serializer.data)
```

## 2.  五个扩展类

### 1）ListModelMixin

列表视图扩展类，提供`list(request, *args, **kwargs)`方法快速实现列表视图，返回200状态码。

该Mixin的list方法会对数据进行过滤和分页。

源代码：

```python
class ListModelMixin(object):
    """
    List a queryset.
    """
    def list(self, request, *args, **kwargs):
        # 过滤
        queryset = self.filter_queryset(self.get_queryset())
		# 分页
        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
		# 序列化
        serializer = self.get_serializer(queryset, many=True)
        return Response(serializer.data)
```

举例：

```python
from rest_framework.mixins import ListModelMixin

class BookListView(ListModelMixin, GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request):
        return self.list(request)
```

### 2）CreateModelMixin

创建视图扩展类，提供`create(request, *args, **kwargs)`方法快速实现创建资源的视图，成功返回201状态码。

如果序列化器对前端发送的数据验证失败，返回400错误。

源代码：

```python
class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        # 获取序列化器
        serializer = self.get_serializer(data=request.data)
        # 验证
        serializer.is_valid(raise_exception=True)
        # 保存
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': str(data[api_settings.URL_FIELD_NAME])}
        except (TypeError, KeyError):
            return {}
```

### 3） RetrieveModelMixin

详情视图扩展类，提供`retrieve(request, *args, **kwargs)`方法，可以快速实现返回一个存在的数据对象。

如果存在，返回200， 否则返回404。

源代码：

```python
class RetrieveModelMixin(object):
    """
    Retrieve a model instance.
    """
    def retrieve(self, request, *args, **kwargs):
        # 获取对象，会检查对象的权限
        instance = self.get_object()
        # 序列化
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
```

举例：

```python
class BookDetailView(RetrieveModelMixin, GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializer

    def get(self, request, pk):
        return self.retrieve(request)
```



### 4）UpdateModelMixin

更新视图扩展类，提供`update(request, *args, **kwargs)`方法，可以快速实现更新一个存在的数据对象。

同时也提供`partial_update(request, *args, **kwargs)`方法，可以实现局部更新。

成功返回200，序列化器校验数据失败时，返回400错误。

源代码：

```python
class UpdateModelMixin(object):
    """
    Update a model instance.
    """
    def update(self, request, *args, **kwargs):
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)

        if getattr(instance, '_prefetched_objects_cache', None):
            # If 'prefetch_related' has been applied to a queryset, we need to
            # forcibly invalidate the prefetch cache on the instance.
            instance._prefetched_objects_cache = {}

        return Response(serializer.data)

    def perform_update(self, serializer):
        serializer.save()

    def partial_update(self, request, *args, **kwargs):
        kwargs['partial'] = True
        return self.update(request, *args, **kwargs)
```

### 5）DestroyModelMixin

删除视图扩展类，提供`destroy(request, *args, **kwargs)`方法，可以快速实现删除一个存在的数据对象。

成功返回204，不存在返回404。

源代码：

```python
class DestroyModelMixin(object):
    """
    Destroy a model instance.
    """
    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        self.perform_destroy(instance)
        return Response(status=status.HTTP_204_NO_CONTENT)

    def perform_destroy(self, instance):
        instance.delete()
```

## 3.  几个可用子类视图

### 1） CreateAPIView

提供 post 方法

继承自： GenericAPIView、CreateModelMixin

### 2）ListAPIView

提供 get 方法

继承自：GenericAPIView、ListModelMixin

### 3）RetireveAPIView

提供 get 方法

继承自:  GenericAPIView、RetrieveModelMixin

### 4）DestoryAPIView

提供 delete 方法

继承自：GenericAPIView、DestoryModelMixin

### 5）UpdateAPIView

提供 put 和 patch 方法

继承自：GenericAPIView、UpdateModelMixin

### 6）RetrieveUpdateAPIView

提供 get、put、patch方法

继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin

### 7）RetrieveUpdateDestoryAPIView

提供 get、put、patch、delete方法

继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin







