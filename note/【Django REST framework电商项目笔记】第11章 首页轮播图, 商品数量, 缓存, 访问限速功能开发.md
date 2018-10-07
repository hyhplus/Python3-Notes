##  首页轮播图接口实现
轮播图接口 & 新品接口 & 下面的系列产品接口  
支付功能开发完了，我们需要进入本地调试环境  
运行一下查看这个本地调试环境是否可以运行。将vue项目中的地址改为我们本地地址。  

banner有一个外键指向我们的goods。  
新建一个配套的BannerSerializer:  

goods/serializers.py:  
```py
class BannerSerializer(serializers.ModelSerializer):
    class Meta:
        model = Banner
        fields = "__all__"
```

goods/views.py:
```py
class BannerViewSet(mixins.ListModelMixin, viewsets.GenericViewSet):
    """
    list:   获取轮播图列表
    """

    queryset = Banner.objects.all().order_by("index")
    serializer_class = BannerSerializer
```

设置url  
```py
# 首页banner轮播图url
router.register(r'banners', BannerViewset, base_name="banners")
```

没有报错但是也没有数据，我们直接来我们的后台管理系统中添加三张轮播图。
![image](images/1.jpg)
![image](https://img-blog.csdn.net/20181005232620652?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1l1eWgxMzE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##  新品功能接口开发
新品是 goods 中有一个 is_new 的选项  
直接使用 Goods 的接口，进行一个过滤，过滤出其中是新品的。为goods的filter添加is_new字段  

goods/filters.py
```py
        fields = ['pricemin', 'pricemax', 'name', 'is_hot','is_new']
```
使用过滤器可以看到url中 is_new =ture  

这时返回200的正确状态码，但是值为空。因为我们并没有给它设置is_new的商品。  

在xadmin后台添加两个新品。  

##  首页商品大分类显示功能
大分类商品开发  

**这个div中元素data之间的关系会复杂一些。**  
一个大类会有很多brand商标。  
大类又有小类，依然一对多  
大类里面的商品。  
加上它还有一张大图片。  

首先来写一个用于这个系列商品展示的Serializer  
```py
class IndexCategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = GoodsCategory
        fields = "__all__"
```
找出其中的一对多关系。  

首先要取到brand。GoodsCategoryBrand中有一个外键指向category，我们为它添加一个related_name。让它反向取的时候比较方便。  
```py
	category = models.ForeignKey(GoodsCategory, on_delete=models.CASCADE, related_name='brands', null=True, blank=True, verbose_name="商品类目")
```
我们就可以通过goods_category反向取到brand。  

首先写一个BrandSerializer  
```py
class BrandSerializer(serializers.ModelSerializer):
    class Meta:
        model = GoodsCategoryBrand
        fields = "__all__"
```
将brand放入IndexCategorySerializer  
```py
# 一个category会有多个brand。
brands = BrandSerializer(many=True) 
```
为什么不能直接用goods的Serializer。  

goods中确实有一个外键指向我们的category。为什么不能反向取。  

因为商品选择哪个类别的时候，往往选择的是最小的类别。(第三类)  
而不是直接从属于第一类。  

外键指向的第三类。而我们要拿到第一类别的数据。直接使用goods的Serializer取不出来。  

```py
goods = serializers.SerializerMethodField()

    def get_goods(self, obj):
        # 将这个商品相关父类子类等都可以进行匹配
        all_goods = Goods.objects.filter(Q(category_id=obj.id)|Q(category__parent_category_id=obj.id)|Q(category__parent_category__parent_category_id=obj.id))
        goods_serializer = GoodsSerializer(all_goods, many=True, context={'request': self.context['request']})
        return goods_serializer.data

```

取到二级商品分类  
```py
    sub_cat = CategorySerializer2(many=True)
```
首页广告商品大图，首先这是一个商品，属于一级类。  

设计IndexAd model来存大类下的广告位。  
```py
class IndexAd(models.Model):
    """
    首页类别标签右边展示的七个商品广告
    """
    category = models.ForeignKey(GoodsCategory, on_delete=models.CASCADE, related_name='category',verbose_name="商品类目")
    goods =models.ForeignKey(Goods, on_delete=models.CASCADE, related_name='goods')

    class Meta:
        verbose_name = '首页广告'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.goods.name

```

```py
ad_goods = serializers.SerializerMethodField()

    def get_ad_goods(self, obj):
        goods_json = {}
        ad_goods = IndexAd.objects.filter(category_id=obj.id, )
        if ad_goods:
            good_ins = ad_goods[0].goods
            goods_json = GoodsSerializer(good_ins, many=False, context={'request': self.context['request']}).data
        return goods_json

```
ad_goods实际上我们只需要它的id和一张图片就够了，不需要使用返回所有商品信息的 
GoodsSerializer，自己取里面的字段做返回。  
或再去写一个Serializer  

编写对应的viewset  
```py
class IndexCategoryViewset(mixins.ListModelMixin, viewsets.GenericViewSet):
    """
    首页商品分类数据
    """
    # 获取已经被我们设置顶部导航的数据
    queryset = GoodsCategory.objects.filter(is_tab=True, name__in=["生鲜食品", "酒水饮料"])
    serializer_class = IndexCategorySerializer

```
这里配置酒水饮料和生鲜食品是因为我们后面添加brand和ad_goods可以挑着这两类添加  

为该接口配置相应的url  
```py
# 首页系列商品展示url
router.register(r'indexgoods', IndexCategoryViewset, base_name="indexgoods")
```
