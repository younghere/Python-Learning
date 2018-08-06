## Django 细节说明：

## 一、字段的参数

### null

该值为True时，Django在数据库用NULL保存空值。默认值为False。对于保存字符串类型数据的字段，请尽量避免将此参数设为True，那样会导致两种‘没有数据’的情况，一种是NULL，另一种是‘空字符串’。

### blank

True时，字段可以为空。默认False。和null参数不同的是，null是纯数据库层面的，而blank是验证相关的，它与表单验证是否允许输入框内为空有关，与数据库无关。所以要小心一个null为False，blank为True的字段接收到一个空值可能会出bug或异常。

### choices

用于页面上的选择框标签，需要先提供一个二维的二元元组，第一个元素表示存在数据库内真实的值，第二个表示页面上显示的具体内容。在浏览器页面上将显示第二个元素的值。例如：

```
    YEAR_IN_SCHOOL_CHOICES = (
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
        ('GR', 'Graduate'),
    )
```

一般来说，最好将选项定义在类里，并取一个直观的名字，如下所示：

```
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```

要获取一个choices的第二元素的值，可以使用`get_FOO_display()`方法，其中的FOO用字段名代替。对于下面的例子：

```
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
    ('S', 'Small'),
    ('M', 'Medium'),
    ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

使用方法：

```
>>> p = Person(name="Fred Flintstone", shirt_size="L")
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

### db_column

该参数用于定义当前字段在数据表内的列名。如果未指定，Django将使用字段名作为列名。

### db_index

该参数接收布尔值。如果为True，数据库将为该字段创建索引。

### db_tablespace

用于字段索引的数据库表空间的名字，前提是当前字段设置了索引。默认值为工程的`DEFAULT_INDEX_TABLESPACE`设置。如果使用的数据库不支持表空间，该参数会被忽略。

### default

字段的默认值，可以是值或者一个可调用对象。如果是可调用对象，那么每次创建新对象时都会调用。设置的默认值不能是一个可变对象，比如列表、集合等等。lambda匿名函数也不可用于default的调用对象，因为匿名函数不能被migrations序列化。

注意：在某种原因不明的情况下将default设置为None，可能会引发`intergyerror：not null constraint failed`，即非空约束失败异常，导致`python manage.py migrate`失败，此时可将None改为False或其它的值，只要不是None就行。

### editable

如果设为False，那么当前字段将不会在admin后台或者其它的ModelForm表单中显示，同时还会被模型验证功能跳过。参数默认值为True。

### error_messages

用于自定义错误信息。参数接收字典类型的值。字典的键可以是`null`、 `blank`、 `invalid`、 `invalid_choice`、 `unique`和`unique_for_date`其中的一个。

### help_text

额外显示在表单部件上的帮助文本。使用时请注意转义为纯文本，防止脚本攻击。

### primary_key

如果你没有给模型的任何字段设置这个参数为True，Django将自动创建一个AutoField自增字段，名为‘id’，并设置为主键。也就是`id = models.AutoField(primary_key=True)`。

如果你为某个字段设置了primary_key=True，则当前字段变为主键，并关闭Django自动生成id主键的功能。

**primary_key=True隐含null=False和unique=True的意思。一个模型中只能有一个主键字段！**

另外，主键字段不可修改，如果你给某个对象的主键赋个新值实际上是创建一个新对象，并不会修改原来的对象。

```
from django.db import models
class Fruit(models.Model):
    name = models.CharField(max_length=100, primary_key=True)
###############    
>>> fruit = Fruit.objects.create(name='Apple')
>>> fruit.name = 'Pear'
>>> fruit.save()
>>> Fruit.objects.values_list('name', flat=True)
['Apple', 'Pear']
```

### unique

设为True时，在整个数据表内该字段的数据不可重复。

注意：对于ManyToManyField和OneToOneField关系类型，该参数无效。

注意： 当unique=True时，db_index参数无须设置，因为unqiue隐含了索引。

注意：自1.11版本后，unique参数可以用于FileField字段。

### unique_for_date

日期唯一。可能不太好理解。举个栗子，如果你有一个名叫title的字段，并设置了参数`unique_for_date="pub_date"`，那么Django将不允许有两个模型对象具备同样的title和pub_date。有点类似联合约束。

### unique_for_month

同上，只是月份唯一。

### unique_for_year

同上，只是年份唯一。

### verbose_name

为字段设置一个人类可读，更加直观的别名。

对于每一个字段类型，除了`ForeignKey`、`ManyToManyField`和`OneToOneField`这三个特殊的关系类型，其第一可选位置参数都是`verbose_name`。如果没指定这个参数，Django会利用字段的属性名自动创建它，并将下划线转换为空格。

下面这个例子的`verbose name`是"person’s first name":

```
first_name = models.CharField("person's first name", max_length=30)
```

下面这个例子的`verbose name`是"first name":

```
first_name = models.CharField(max_length=30)
```

对于外键、多对多和一对一字字段，由于第一个参数需要用来指定关联的模型，因此必须用关键字参数`verbose_name`来明确指定。如下：

```
poll = models.ForeignKey(
    Poll,
    on_delete=models.CASCADE,
    verbose_name="the related poll",
    )
sites = models.ManyToManyField(Site, verbose_name="list of sites")
    place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    verbose_name="related place",
)
```

另外，你无须大写`verbose_name`的首字母，Django自动为你完成这一工作。

### validators

运行在该字段上的验证器的列表。二、字段类型

## 二、普通字段

| 类型                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| AutoField                  | 一个自动增加的整数类型字段。通常你不需要自己编写它，Django会自动帮你添加字段：`id = models.AutoField(primary_key=True)`，这是一个自增字段，从1开始计数。如果你非要自己设置主键，那么请务必将字段设置为`primary_key=True`。Django在一个模型中只允许有一个自增字段，并且该字段必须为主键！ |
| BigAutoField               | (1.10新增)64位整数类型自增字段，数字范围更大，从1到9223372036854775807 |
| BigIntegerField            | 64位整数字段（看清楚，非自增），类似IntegerField ，-9223372036854775808 到9223372036854775807。在Django的模板表单里体现为一个textinput标签。 |
| BinaryField                | 二进制数据类型。使用受限，少用。                             |
| **BooleanField**           | 布尔值类型。默认值是None。在HTML表单中体现为CheckboxInput标签。如果要接收null值，请使用NullBooleanField。 |
| **CharField**              | 字符串类型。必须接收一个max_length参数，表示字符串长度不能超过该值。默认的表单标签是input text。最常用的filed，没有之一！ |
| CommaSeparatedIntegerField | 逗号分隔的整数类型。必须接收一个max_length参数。常用于表示较大的金额数目，例如1,000,000元。 |
| **DateField**              | `class DateField(auto_now=False, auto_now_add=False, **options)`日期类型。一个Python中的datetime.date的实例。在HTML中表现为TextInput标签。在admin后台中，Django会帮你自动添加一个JS的日历表和一个“Today”快捷方式，以及附加的日期合法性验证。两个重要参数：（参数互斥，不能共存） `auto_now`:每当对象被保存时将字段设为当前日期，常用于保存最后修改时间。`auto_now_add`：每当对象被创建时，设为当前日期，常用于保存创建日期(注意，它是不可修改的)。设置上面两个参数就相当于给field添加了`editable=False`和`blank=True`属性。如果想具有修改属性，请用default参数。例子：`pub_time = models.DateField(auto_now_add=True)`，自动添加发布时间。 |
| DateTimeField              | 日期时间类型。Python的datetime.datetime的实例。与DateField相比就是多了小时、分和秒的显示，其它功能、参数、用法、默认值等等都一样。 |
| DecimalField               | 固定精度的十进制小数。相当于Python的Decimal实例，必须提供两个指定的参数！参数`max_digits`：最大的位数，必须大于或等于小数点位数 。`decimal_places`：小数点位数，精度。 当`localize=False`时，它在HTML表现为NumberInput标签，否则是text类型。例子：储存最大不超过999，带有2位小数位精度的数，定义如下：`models.DecimalField(..., max_digits=5, decimal_places=2)`。 |
| DurationField              | 持续时间类型。存储一定期间的时间长度。类似Python中的timedelta。在不同的数据库实现中有不同的表示方法。常用于进行时间之间的加减运算。但是小心了，这里有坑，PostgreSQL等数据库之间有兼容性问题！ |
| **EmailField**             | 邮箱类型，默认max_length最大长度254位。使用这个字段的好处是，可以使用DJango内置的EmailValidator进行邮箱地址合法性验证。 |
| **FileField**              | `class FileField(upload_to=None, max_length=100, **options)`上传文件类型，后面单独介绍。 |
| FilePathField              | 文件路径类型，后面单独介绍                                   |
| FloatField                 | 浮点数类型，参考整数类型                                     |
| **ImageField**             | 图像类型，后面单独介绍。                                     |
| **IntegerField**           | 整数类型，最常用的字段之一。取值范围-2147483648到2147483647。在HTML中表现为NumberInput标签。 |
| **GenericIPAddressField**  | `class GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)[source]`,IPV4或者IPV6地址，字符串形式，例如`192.0.2.30`或者`2a02:42fe::4`在HTML中表现为TextInput标签。参数`protocol`默认值为‘both’，可选‘IPv4’或者‘IPv6’，表示你的IP地址类型。 |
| NullBooleanField           | 类似布尔字段，只不过额外允许`NULL`作为选项之一。             |
| PositiveIntegerField       | 正整数字段，包含0,最大2147483647。                           |
| PositiveSmallIntegerField  | 较小的正整数字段，从0到32767。                               |
| SlugField                  | slug是一个新闻行业的术语。一个slug就是一个某种东西的简短标签，包含字母、数字、下划线或者连接线，通常用于URLs中。可以设置max_length参数，默认为50。 |
| SmallIntegerField          | 小整数，包含-32768到32767。                                  |
| **TextField**              | 大量文本内容，在HTML中表现为Textarea标签，最常用的字段类型之一！如果你为它设置一个max_length参数，那么在前端页面中会受到输入字符数量限制，然而在模型和数据库层面却不受影响。只有CharField才能同时作用于两者。 |
| TimeField                  | 时间字段，Python中datetime.time的实例。接收同DateField一样的参数，只作用于小时、分和秒。 |
| **URLField**               | 一个用于保存URL地址的字符串类型，默认最大长度200。           |
| **UUIDField**              | 用于保存通用唯一识别码（Universally Unique Identifier）的字段。使用Python的UUID类。在PostgreSQL数据库中保存为uuid类型，其它数据库中为char(32)。这个字段是自增主键的最佳替代品，后面有例子展示。 |

**这里有如何上传文件和图片的方法：**

#### 1.**FileField：**

```
class FileField(upload_to=None, max_length=100, **options)[source]
```

上传文件字段（不能设置为主键）。默认情况下，该字段在HTML中表现为一个ClearableFileInput标签。在数据库内，我们实际保存的是一个字符串类型，默认最大长度100，可以通过max_length参数自定义。真实的文件是保存在服务器的文件系统内的。

重要参数`upload_to`用于设置上传地址的目录和文件名。如下例所示：

```python
class MyModel(models.Model):
    # 文件被传至`MEDIA_ROOT/uploads`目录，MEDIA_ROOT由你在settings文件中设置
    upload = models.FileField(upload_to='uploads/')
    # 或者
    # 被传到`MEDIA_ROOT/uploads/2015/01/30`目录，增加了一个时间划分
    upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
```

**Django很人性化地帮我们实现了根据日期生成目录的方式！**

**upload_to参数也可以接收一个回调函数，该函数返回具体的路径字符串**，如下例：

```python
def user_directory_path(instance, filename):
    #文件上传到MEDIA_ROOT/user_<id>/<filename>目录中
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
```

例子中，`user_directory_path`这种回调函数，必须接收两个参数，然后返回一个Unix风格的路径字符串。参数`instace`代表一个定义了`FileField`的模型的实例，说白了就是当前数据记录。`filename`是原本的文件名。

#### 2. **ImageField**

```
class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)[source]
```

用于保存图像文件的字段。其基本用法和特性与FileField一样，只不过多了两个属性height和width。默认情况下，该字段在HTML中表现为一个ClearableFileInput标签。在数据库内，我们实际保存的是一个字符串类型，默认最大长度100，可以通过max_length参数自定义。真实的图片是保存在服务器的文件系统内的。

`height_field`参数：保存有图片高度信息的模型字段名。
`width_field`参数：保存有图片宽度信息的模型字段名。

**使用Django的ImageField需要提前安装pillow模块，pip install pillow即可。**

使用FileField或者ImageField字段的步骤：

1. 在settings文件中，配置`MEDIA_ROOT`，作为你上传文件在服务器中的基本路径（为了性能考虑，这些文件不会被储存在数据库中）。再配置个`MEDIA_URL`，作为公用URL，指向上传文件的基本路径。请确保Web服务器的用户账号对该目录具有写的权限。
2. 添加FileField或者ImageField字段到你的模型中，定义好`upload_to`参数，文件最终会放在`MEDIA_ROOT`目录的“upload_to”子目录中。
3. 所有真正被保存在数据库中的，只是指向你上传文件路径的字符串而已。可以通过url属性，在Django的模板中方便的访问这些文件。例如，假设你有一个ImageField字段，名叫`mug_shot`，那么在Django模板的HTML文件中，可以使用`{{ object.mug_shot.url }}`来获取该文件。其中的object用你具体的对象名称代替。
4. 可以通过`name`和`size`属性，获取文件的名称和大小信息。

**安全建议:**

无论你如何保存上传的文件，一定要注意他们的内容和格式，避免安全漏洞！务必对所有的上传文件进行安全检查，确保它们不出问题！如果你不加任何检查就盲目的让任何人上传文件到你的服务器文档根目录内，比如上传了一个CGI或者PHP脚本，很可能就会被访问的用户执行，这具有致命的危害。

#### 3. **FilePathField**

```
class FilePathField(path=None, match=None, recursive=False, max_length=100, **options)[source]
```

一种用来保存文件路径信息的字段。在数据表内以字符串的形式存在，默认最大长度100，可以通过max_length参数设置。

它包含有下面的一些参数：

`path`：必须指定的参数。表示一个系统绝对路径。

`match`:可选参数，一个正则表达式，用于过滤文件名。只匹配基本文件名，不匹配路径。例如`foo.*\.txt$`，只匹配文件名`foo23.txt`，不匹配`bar.txt`与`foo23.png`。

`recursive`:可选参数，只能是True或者False。默认为False。决定是否包含子目录，也就是是否递归的意思。

`allow_files`:可选参数，只能是True或者False。默认为True。决定是否应该将文件名包括在内。它和`allow_folders`其中，必须有一个为True。

`allow_folders`： 可选参数，只能是True或者False。默认为False。决定是否应该将目录名包括在内。

比如：

```
FilePathField(path="/home/images", match="foo.*", recursive=True)
```

它只匹配`/home/images/foo.png`，但不匹配`/home/images/foo/bar.png`，因为默认情况，只匹配文件名，而不管路径是怎么样的。

#### 4. **UUIDField：**

数据库无法自己生成uuid，因此需要如下使用default参数：

```
import uuid     # Python的内置模块
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # 其它字段 
```

## 三、关系字段

### 1.多对一（ForeignKey）

多对一的关系，通常被称为外键。外键字段类的定义如下：

```
class ForeignKey(to, on_delete, **options)[source]
```

外键需要两个位置参数，一个是关联的模型，另一个是`on_delete`选项。实际上，在目前版本中，`on_delete`选项也可以不设置，但Django极力反对如此，因此在Django2.0版本后，该选项会设置为必填。

**外键要定义在‘多’的一方！**

```
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    # ...

class Manufacturer(models.Model):
    # ...
    pass
```

上面的例子中，每辆车都会有一个生产工厂，一个工厂可以生产N辆车，于是用一个外键字段manufacturer表示，并放在Car模型中。注意，此manufacturer非彼Manufacturer模型类，它是一个字段的名称。在Django的模型定义中，经常出现类似的英文单词大小写不同，一定要注意区分！

如果要关联的对象在另外一个app中，可以显式的指出。下例假设Manufacturer模型存在于production这个app中，则Car模型的定义如下：

```
class Car(models.Model):
    manufacturer = models.ForeignKey(
        'production.Manufacturer',      # 关键在这里！！
        on_delete=models.CASCADE,
    )
```

如果要创建一个递归的外键，也就是自己关联自己的的外键，使用下面的方法：

```
models.ForeignKey('self', on_delete=models.CASCADE)
```

核心在于‘self’这个引用。什么时候需要自己引用自己的外键呢？典型的例子就是评论系统！一条评论可以被很多人继续评论，如下所示：

```
class Comment(models.Model):
    title = models.CharField(max_length=128)
    text = models.TextField()
    parent_comment = models.ForeignKey('self', on_delete=models.CASCADE)
    # .....
```

注意上面的外键字段定义的是父评论，而不是子评论。为什么呢？因为外键要放在‘多’的一方！

在实际的数据库后台，Django会为每一个外键添加`_id`后缀，并以此创建数据表里的一列。在上面的工厂与车的例子中，Car模型对应的数据表中，会有一列叫做`manufacturer_id`。但实际上，在Django代码中你不需要使用这个列名，除非你书写原生的SQL语句，一般我们都直接使用字段名`manufacturer`。

关系字段的定义还有个小坑。在后面我们会讲到的`verbose_name`参数用于设置字段的别名。很多情况下，为了方便，我们都会设置这么个值，并且作为字段的第一位置参数。但是对于关系字段，其第一位置参数永远是关系对象，不能是`verbose_name`，一定要注意！

#### 参数说明：

外键还有一些重要的参数，说明如下：

#### on_delete

当一个被外键关联的对象被删除时，Django将模仿`on_delete`参数定义的SQL约束执行相应操作。比如，你有一个可为空的外键，并且你想让它在关联的对象被删除时，自动设为null，可以如下定义：

```
user = models.ForeignKey(
    User,
    models.SET_NULL,
    blank=True,
    null=True,
)
```

该参数可选的值都内置在`django.db.models`中，包括：

- CASCADE：模拟SQL语言中的`ON DELETE CASCADE`约束，将定义有外键的模型对象同时删除！（该操作为当前Django版本的默认操作！）
- PROTECT:阻止上面的删除操作，但是弹出`ProtectedError`异常
- SET_NULL：将外键字段设为null，只有当字段设置了`null=True`时，方可使用该值。
- SET_DEFAULT:将外键字段设为默认值。只有当字段设置了default参数时，方可使用。
- DO_NOTHING：什么也不做。
- SET()：设置为一个传递给SET()的值或者一个回调函数的返回值。注意大小写。

```
from django.conf import settings
from django.contrib.auth import get_user_model
from django.db import models

def get_sentinel_user():
    return get_user_model().objects.get_or_create(username='deleted')[0]

class MyModel(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.SET(get_sentinel_user),
    )
```

#### limit_choices_to

该参数用于限制外键所能关联的对象，只能用于Django的ModelForm（Django的表单模块）和admin后台，对其它场合无限制功能。其值可以是一个字典、Q对象或者一个返回字典或Q对象的函数调用，如下例所示：

```
staff_member = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    limit_choices_to={'is_staff': True},
)
```

这样定义，则ModelForm的`staff_member`字段列表中，只会出现那些`is_staff=True`的Users对象，这一功能对于admin后台非常有用。

可以参考下面的方式，使用函数调用：

```
def limit_pub_date_choices():
    return {'pub_date__lte': datetime.date.utcnow()}

# ...
limit_choices_to = limit_pub_date_choices
# ...
```

#### related_name

用于关联对象反向引用模型的名称。以前面车和工厂的例子解释，就是从工厂反向关联到车的关系名称。

通常情况下，这个参数我们可以不设置，Django会默认以模型的小写作为反向关联名，比如对于工厂就是`car`，如果你觉得`car`还不够直观，可以如下定义：

```
class Car(models.Model):
    manufacturer = models.ForeignKey(
        'production.Manufacturer',      
        on_delete=models.CASCADE,
        related_name='car_producted_by_this_manufacturer',  # 看这里！！
    )
```

也许我定义了一个蹩脚的词，但表达的意思很清楚。以后从工厂对象反向关联到它所生产的汽车，就可以使用`maufacturer.car_producted_by_this_manufacturer`了。

如果你不想为外键设置一个反向关联名称，可以将这个参数设置为“+”或者以“+”结尾，如下所示：

```
user = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name='+',
)
```

#### related_query_name

反向关联查询名。用于从目标模型反向过滤模型对象的名称。（过滤和查询在后续章节会介绍）

```
class Tag(models.Model):
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name="tags",
        related_query_name="tag",       # 注意这一行
    )
    name = models.CharField(max_length=255)

# 现在可以使用‘tag’作为查询名了
Article.objects.filter(tag__name="important")
```

#### to_field

默认情况下，外键都是关联到被关联对象的主键上（一般为id）。如果指定这个参数，可以关联到指定的字段上，但是该字段必须具有`unique=True`属性，也就是具有唯一属性。

#### db_constraint

默认情况下，这个参数被设为True，表示遵循数据库约束，这也是大多数情况下你的选择。如果设为False，那么将无法保证数据的完整性和合法性。在下面的场景中，你可能需要将它设置为False：

- 有历史遗留的不合法数据，没办法的选择
- 你正在分割数据表

当它为False，并且你试图访问一个不存在的关系对象时，会抛出DoesNotExist 异常。

#### swappable

控制迁移框架的动作，如果当前外键指向一个可交换的模型。使用场景非常稀少，通常请将该参数保持默认的True。

### 2.多对多（ManyToManyField）

多对多关系在数据库中也是非常常见的关系类型。比如一本书可以有好几个作者，一个作者也可以写好几本书。多对多的字段可以定义在任何的一方，请尽量定义在符合人们思维习惯的一方，但不要同时都定义。

```
class ManyToManyField(to, **options)[source]
```

多对多关系需要一个位置参数：关联的对象模型。它的用法和外键多对一基本类似。

**在数据库后台，Django实际上会额外创建一张用于体现多对多关系的中间表**。默认情况下，该表的名称是“多对多字段名+关联对象模型名+一个独一无二的哈希码”，例如‘author_books_9cdf4’，当然你也可以通过`db_table`选项，自定义表名。

#### 参数说明：

##### related_name

参考外键的相同参数。

##### related_query_name

参考外键的相同参数。

##### limit_choices_to

参考外键的相同参数。但是对于使用`through`参数自定义中间表的多对多字段无效。

##### symmetrical

默认情况下，Django中的多对多关系是对称的。看下面的例子：

```
from django.db import models

class Person(models.Model):
    friends = models.ManyToManyField("self")
```

Django认为，如果我是你的朋友，那么你也是我的朋友，这是一种对称关系，Django不会为Person模型添加`person_set`属性用于反向关联。如果你不想使用这种对称关系，可以将symmetrical设置为False，这将强制Django为反向关联添加描述符。

##### through

###### （定义中间表）

如果你想自定义多对多关系的那张额外的关联表，可以使用这个参数！参数的值为一个中间模型。

最常见的使用场景是你需要为多对多关系添加额外的数据，比如两个人建立QQ好友的时间。

通常情况下，这张表在数据库内的结构是这个样子的：

```
中间表的id列....模型对象的id列.....被关联对象的id列
# 各行数据
```

如果自定义中间表并添加时间字段，则在数据库内的表结构如下：

```
中间表的id列....模型对象的id列.....被关联对象的id列.....时间对象列
# 各行数据
```

看下面的例子：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',       ## 自定义中间表
        through_fields=('group', 'person'),
    )

class Membership(models.Model):  # 这就是具体的中间表模型
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```

上面的代码中，通过`class Membership(models.Model)`定义了一个新的模型，用来保存Person和Group模型的多对多关系，并且同时增加了‘邀请人’和‘邀请时间’的字段。

**through参数在某些使用场景中是必须的，至关重要，请务必掌握！**

##### through_fields

接着上面的例子。Membership模型中包含两个关联Person的外键，Django无法确定到底使用哪个作为和Group关联的对象。所以，在这个例子中，必须显式的指定`through_fields`参数，用于定义关系。

`through_fields`参数接收一个二元元组('field1', 'field2')，field1是指向定义有多对多关系的模型的外键字段的名称，这里是Membership中的‘group’字段（注意大小写），另外一个则是指向目标模型的外键字段的名称，这里是Membership中的‘person’，而不是‘inviter’。

再通俗的说，就是`through_fields`参数指定从中间表模型Membership中选择哪两个字段，作为关系连接字段。

##### db_table

设置中间表的名称。不指定的话，则使用默认值。

##### db_constraint

参考外键的相同参数。

##### swappable

参考外键的相同参数。

**ManyToManyField多对多字段不支持Django内置的validators验证功能。**

**null参数对ManyToManyField多对多字段无效！设置null=True毫无意义**

### 3.一对一（OneToOneField）

一对一关系类型的定义如下：

```
class OneToOneField(to, on_delete, parent_link=False, **options)[source]
```

从概念上讲，一对一关系非常类似具有`unique=True`属性的外键关系，但是反向关联对象只有一个。这种关系类型多数用于当一个模型需要从别的模型扩展而来的情况。比如，Django自带auth模块的User用户表，如果你想在自己的项目里创建用户模型，又想方便的使用Django的认证功能，那么一个比较好的方案就是在你的用户模型里，使用一对一关系，添加一个与auth模块User模型的关联字段。

该关系的第一位置参数为关联的模型，其用法和前面的多对一外键一样。

如果你没有给一对一关系设置`related_name`参数，Django将使用当前模型的小写名作为默认值。

看下面的例子：

```
from django.conf import settings
from django.db import models

# 两个字段都使用一对一关联到了Django内置的auth模块中的User模型
class MySpecialUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    supervisor = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='supervisor_of',
    )
```

这样下来，你的User模型将拥有下面的属性：

```
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')
True
>>> hasattr(user, 'supervisor_of')
True
```

OneToOneField一对一关系拥有和多对一外键关系一样的额外可选参数，只是多了一个parent_link参数。

------

跨模块的模型：

有时候，我们关联的模型并不在当前模型的文件内，没关系，就像我们导入第三方库一样的从别的模块内导入进来就好，如下例所示：

```
from django.db import models
from geography.models import ZipCode

class Restaurant(models.Model):
    # ...
    zip_code = models.ForeignKey(
        ZipCode,
        on_delete=models.SET_NULL,
        blank=True,
        null=True,
    )
```

## 三、模型的元数据Meta

模型的元数据，指的是“除了字段外的所有内容”，例如排序方式、数据库表名、人类可读的单数或者复数名等等。所有的这些都是非必须的，甚至元数据本身对模型也是非必须的。但是，我要说但是，有些元数据选项能给予你极大的帮助，在实际使用中具有重要的作用，是实际应用的‘必须’。

想在模型中增加元数据，方法很简单，在模型类中添加一个子类，名字是固定的`Meta`，然后在这个Meta类下面增加各种元数据选项或者说设置项。参考下面的例子：

```
from django.db import models

class Ox(models.Model):
    horn_length = models.IntegerField()

    class Meta:         # 注意，是模型的子类，要缩进！
        ordering = ["horn_length"]
        verbose_name_plural = "oxen"
```

上面的例子中，我们为模型Ox增加了两个元数据‘ordering’和‘verbose_name_plural’，分别表示排序和复数名，下面我们会详细介绍有哪些可用的元数据选项。

**强调：每个模型都可以有自己的元数据类，每个元数据类也只对自己所在模型起作用。**

------

### abstract

如果`abstract=True`，那么模型会被认为是一个抽象模型。抽象模型本身不实际生成数据库表，而是作为其它模型的父类，被继承使用。具体内容可以参考Django模型的继承。

------

### app_label

如果定义了模型的app没有在`INSTALLED_APPS`中注册，则必须通过此元选项声明它属于哪个app，例如：

```
app_label = 'myapp'
```

------

### base_manager_name

自定义模型的`_base_manager`管理器的名字。模型管理器是Django为模型提供的API所在。Django1.10新增。

------

### db_table

指定在数据库中，当前模型生成的数据表的表名。比如：

```
db_table = 'my_freinds'
```

友情建议：使用MySQL数据库时，`db_table`用小写英文。

------

### db_tablespace

自定义数据库表空间的名字。默认值是工程的`DEFAULT_TABLESPACE`设置。

------

### default_manager_name

自定义模型的`_default_manager`管理器的名字。Django1.10新增。

------

### default_related_name

默认情况下，从一个模型反向关联设置有关系字段的源模型，我们使用`<model_name>_set`，也就是源模型的名字+下划线+`set`。

这个元数据选项可以让你自定义反向关系名，同时也影响反向查询关系名！看下面的例子：

```
from django.db import models

class Foo(models.Model):
    pass

class Bar(models.Model):
    foo = models.ForeignKey(Foo)

    class Meta:
        default_related_name = 'bars'   # 关键在这里
```

具体的使用差别如下：

```
>>> bar = Bar.objects.get(pk=1)
>>> # 不能再使用"bar"作为反向查询的关键字了。
>>> Foo.objects.get(bar=bar)
>>> # 而要使用你自己定义的"bars"了。
>>> Foo.objects.get(bars=bar)
```

------

### get_latest_by

Django管理器给我们提供有latest()和earliest()方法，分别表示获取最近一个和最前一个数据对象。但是，如何来判断最近一个和最前面一个呢？也就是根据什么来排序呢？

`get_latest_by`元数据选项帮你解决这个问题，它可以指定一个类似 `DateField`、`DateTimeField`或者`IntegerField`这种可以排序的字段，作为latest()和earliest()方法的排序依据，从而得出最近一个或最前面一个对象。例如：

```
get_latest_by = "order_date"
```

------

### managed

该元数据默认值为True，表示Django将按照既定的规则，管理数据库表的生命周期。

如果设置为False，将不会针对当前模型创建和删除数据库表。在某些场景下，这可能有用，但更多时候，你可以忘记该选项。

------

### order_with_respect_to

这个选项不好理解。其用途是根据指定的字段进行排序，通常用于关系字段。看下面的例子：

```
from django.db import models

class Question(models.Model):
    text = models.TextField()
    # ...

class Answer(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # ...

    class Meta:
        order_with_respect_to = 'question'
```

上面在Answer模型中设置了`order_with_respect_to = 'question'`，这样的话，Django会自动提供两个API，`get_RELATED_order()`和`set_RELATED_order()`，其中的`RELATED`用小写的模型名代替。假设现在有一个Question对象，它关联着多个Answer对象，下面的操作返回包含关联的Anser对象的主键的列表[1,2,3]：

```
>>> question = Question.objects.get(id=1)
>>> question.get_answer_order()
[1, 2, 3]
```

我们可以通过`set_RELATED_order()`方法，指定上面这个列表的顺序：

```
>>> question.set_answer_order([3, 1, 2])
```

同样的，关联的对象也获得了两个方法`get_next_in_order()`和`get_previous_in_order()`，用于通过特定的顺序访问对象，如下所示：

```
>>> answer = Answer.objects.get(id=2)
>>> answer.get_next_in_order()
<Answer: 3>
>>> answer.get_previous_in_order()
<Answer: 1>
```

这个元数据的作用......还没用过，囧。

------

### ordering

最常用的元数据之一了！

用于指定该模型生成的所有对象的排序方式，接收一个字段名组成的元组或列表。默认按升序排列，如果在字段名前加上字符“-”则表示按降序排列，如果使用字符问号“？”表示随机排列。请看下面的例子：

```
ordering = ['pub_date']             # 表示按'pub_date'字段进行升序排列
ordering = ['-pub_date']            # 表示按'pub_date'字段进行降序排列
ordering = ['-pub_date', 'author']  # 表示先按'pub_date'字段进行降序排列，再按`author`字段进行升序排列。
```

------

### permissions

该元数据用于当创建对象时增加额外的权限。它接收一个所有元素都是二元元组的列表或元组，每个元素都是`(权限代码, 直观的权限名称)`的格式。比如下面的例子：

```
permissions = (("can_deliver_pizzas", "可以送披萨"),)
```

------

### default_permissions

Django默认给所有的模型设置('add', 'change', 'delete')的权限，也就是增删改。你可以自定义这个选项，比如设置为一个空列表，表示你不需要默认的权限，但是这一操作必须在执行migrate命令之前。

------

### proxy

如果设置了`proxy = True`，表示使用代理模式的模型继承方式。具体内容与abstract选项一样，参考模型继承章节。

------

### required_db_features

声明模型依赖的数据库功能。比如['gis_enabled']，表示模型的建立依赖GIS功能。

------

### required_db_vendor

声明模型支持的数据库。Django默认支持`sqlite, postgresql, mysql, oracle`。

------

### select_on_save

决定是否使用1.6版本之前的`django.db.models.Model.save()`算法保存对象。默认值为False。这个选项我们通常不用关心。

------

### indexes

Django1.11新增的选项。

接收一个应用在当前模型上的索引列表，如下例所示：

```
from django.db import models

class Customer(models.Model):
    first_name = models.CharField(max_length=100)
    last_name = models.CharField(max_length=100)

    class Meta:
        indexes = [
            models.Index(fields=['last_name', 'first_name']),
            models.Index(fields=['first_name'], name='first_name_idx'),
        ]
```

------

### unique_together

这个元数据是非常重要的一个！它等同于数据库的联合约束！

举个例子，假设有一张用户表，保存有用户的姓名、出生日期、性别和籍贯等等信息。要求是所有的用户唯一不重复，可现在有好几个叫“张伟”的，如何区别它们呢？（不要和我说主键唯一，这里讨论的不是这个问题）

我们可以设置不能有两个用户在同一个地方同一时刻出生并且都叫“张伟”，使用这种联合约束，保证数据库能不能重复添加用户（也不要和我谈小概率问题）。在Django的模型中，如何实现这种约束呢？

使用`unique_together`，也就是联合唯一！

比如：

```
unique_together = (('name', 'birth_day', 'address'),)
```

这样，哪怕有两个在同一天出生的张伟，但他们的籍贯不同，也就是两个不同的用户。一旦三者都相同，则会被Django拒绝创建。这一元数据经常被用在admin后台，并且强制应用于数据库层面。

unique_together接收一个二维的元组((xx,xx,xx,...),(),(),()...)，每一个元素都是一个元组，表示一组联合唯一约束，可以同时设置多组约束。为了方便，对于只有一组约束的情况下，可以简单地使用一维元素，例如：

```
unique_together = ('name', 'birth_day', 'address')
```

联合唯一无法作用于普通的多对多字段。

------

### index_together

即将废弃，使用`index`元数据代替。

------

### verbose_name

最常用的元数据之一！用于设置模型对象的直观、人类可读的名称。可以用中文。例如：

```
verbose_name = "story"
verbose_name = "披萨"
```

如果你不指定它，那么Django会使用小写的模型名作为默认值。

------

### verbose_name_plural

英语有单数和复数形式。这个就是模型对象的复数名，比如“apples”。因为我们中文通常不区分单复数，所以保持和`verbose_name`一致也可以。

```
verbose_name_plural = "stories"
verbose_name_plural = "披萨"
```

如果不指定该选项，那么默认的复数名字是`verbose_name`加上‘s’

------

### label

前面介绍的元数据都是可修改和设置的，但还有两个只读的元数据，label就是其中之一。

label等同于`app_label.object_name`。例如`polls.Question`，polls是应用名，Question是模型名。

------

### label_lower

同上，不过是小写的模型名。

## 四、多对多中间表详解

### 1.默认中间表

首先，模型是这样的：

```
class Person(models.Model):
    name = models.CharField(max_length=128)

    def __str__(self):
        return self.name


class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person)

    def __str__(self):
        return self.name
```

在Group模型中，通过members字段，以ManyToMany方式与Person模型建立了关系。

让我们到数据库内看一下实际的内容，Django为我们创建了三张数据表，其中的app1是应用名。

![image.png-6.4kB](http://static.zybuluo.com/feixuelove1009/ce9q6hq63vz6769bg5qe85id/image.png)

然后我在数据库中添加了下面的Person对象：

![image.png-16.7kB](http://static.zybuluo.com/feixuelove1009/ygwped2iv1l14jm1mlz3otsd/image.png)

再添加下面的Group对象：

![image.png-13.9kB](http://static.zybuluo.com/feixuelove1009/9dhr7a33ixgyl8qimq5reyth/image.png)

让我们来看看，中间表是个什么样子的：

![image.png-23.7kB](http://static.zybuluo.com/feixuelove1009/s02nszzgk1h824gg32qg7zok/image.png)

首先有一列id，这是Django默认添加的，没什么好说的。然后是Group和Person的id列，这是默认情况下，Django关联两张表的方式。如果你要设置关联的列，可以使用to_field参数。

可见在中间表中，并不是将两张表的数据都保存在一起，而是通过id的关联进行映射。

### 2.自定义中间表

一般情况，普通的多对多已经够用，无需自己创建第三张关系表。但是某些情况可能更复杂一点，比如如果你想保存某个人加入某个分组的时间呢？想保存进组的原因呢？

Django提供了一个`through`参数，用于指定中间模型，你可以将类似进组时间，邀请原因等其他字段放在这个中间模型内。例子如下：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=128)
    def __str__(self): 
        return self.name
        
class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(Person, through='Membership')
    def __str__(self): 
        return self.name

class Membership(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    date_joined = models.DateField()        # 进组时间
    invite_reason = models.CharField(max_length=64)  # 邀请原因
```

在中间表中，我们至少要编写两个外键字段，分别指向关联的两个模型。在本例中就是‘Person’和‘group’。
这里，我们额外增加了‘date_joined’字段，用于保存人员进组的时间，‘invite_reason’字段用于保存邀请进组的原因。

下面我们依然在数据库中实际查看一下（应用名为app2）：

![image.png-3.8kB](http://static.zybuluo.com/feixuelove1009/93oeamp9p6cybhwke0nnhtr7/image.png)

注意中间表的名字已经变成“app2_membership”了。

![image.png-16.5kB](http://static.zybuluo.com/feixuelove1009/5tmg0nekzfc2gv4dteixw6ja/image.png)

![image.png-13.8kB](http://static.zybuluo.com/feixuelove1009/o2mr6uco7xnsyvoq0bq40m83/image.png)

Person和Group没有变化。

![image.png-42.6kB](http://static.zybuluo.com/feixuelove1009/wqzlms0kni4hz91x8bxjcou3/image.png)

但是中间表就截然不同了！它完美的保存了我们需要的内容。

### 3.使用中间表

针对上面的中间表，下面是一些使用例子（以欧洲著名的甲壳虫乐队成员为例）：

```
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles,
... date_joined=date(1962, 8, 16),
... invite_reason="Needed a new drummer.")
>>> m1.save()
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>]>
>>> ringo.group_set.all()
<QuerySet [<Group: The Beatles>]>
>>> m2 = Membership.objects.create(person=paul, group=beatles,
... date_joined=date(1960, 8, 1),
... invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
```

与普通的多对多不一样，使用自定义中间表的多对多不能使用add(), create(),remove(),和set()方法来创建、删除关系，看下面：

```
>>> # 无效
>>> beatles.members.add(john)
>>> # 无效
>>> beatles.members.create(name="George Harrison")
>>> # 无效
>>> beatles.members.set([john, paul, ringo, george])
```

为什么？因为上面的方法无法提供加入时间、邀请原因等中间模型需要的字段内容。唯一的办法只能是通过创建中间模型的实例来创建这种类型的多对多关联。但是，clear()方法是有效的，它能清空所有的多对多关系。

```
>>> # 甲壳虫乐队解散了
>>> beatles.members.clear()
>>> # 删除了中间模型的对象
>>> Membership.objects.all()
<QuerySet []>
```

一旦你通过创建中间模型实例的方法建立了多对多的关联，你立刻就可以像普通的多对多那样进行查询操作：

```
# 查找组内有Paul这个人的所有的组（以Paul开头的名字）
>>> Group.objects.filter(members__name__startswith='Paul')
<QuerySet [<Group: The Beatles>]>
```

可以使用中间模型的属性进行查询：

```
# 查找甲壳虫乐队中加入日期在1961年1月1日之后的成员
>>> Person.objects.filter(
... group__name='The Beatles',
... membership__date_joined__gt=date(1961,1,1))
<QuerySet [<Person: Ringo Starr]>
```

可以像普通模型一样使用中间模型：

```
>>> ringos_membership = Membership.objects.get(group=beatles, person=ringo)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
>>> ringos_membership = ringo.membership_set.get(group=beatles)
>>> ringos_membership.date_joined
datetime.date(1962, 8, 16)
>>> ringos_membership.invite_reason
'Needed a new drummer.'
```

这一部分内容，需要结合后面的模型query，如果暂时看不懂，没有关系。

------

对于中间表，有一点要注意（在前面章节已经介绍过，再次重申一下），默认情况下，中间模型只能包含一个指向源模型的外键关系，上面例子中，也就是在Membership中只能有Person和Group外键关系各一个，不能多。否则，你必须显式的通过`ManyToManyField.through_fields`参数指定关联的对象。参考下面的例子：

```
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
    Person,
    through='Membership',
    through_fields=('group', 'person'),
    )

class Membership(models.Model):
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
    Person,
    on_delete=models.CASCADE,
    related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```

## 五、模型的继承

很多时候，我们都不是从‘一穷二白’开始编写模型的，有时候可以从第三方库中继承，有时候可以从以前的代码中继承，甚至现写一个模型用于被其它模型继承。这样做的好处，我就不赘述了，每个学习Django的人都非常清楚。

类同于Python的类继承，Django也有完善的继承机制。

Django中所有的模型都必须继承`django.db.models.Model`模型，不管是直接继承也好，还是间接继承也罢。

你唯一需要决定的是，父模型是否是一个独立自主的，同样在数据库中创建数据表的模型，还是一个只用来保存子模型共有内容，并不实际创建数据表的抽象模型。

Django有三种继承的方式：

- 抽象基类：被用来继承的模型被称为`Abstract base classes`，将子类共同的数据抽离出来，供子类继承重用，它不会创建实际的数据表；
- 多表继承：`Multi-table inheritance`，每一个模型都有自己的数据库表；
- 代理模型：如果你只想修改模型的Python层面的行为，并不想改动模型的字段，可以使用代理模型。

**注意！同Python的继承一样，Django也是可以同时继承两个以上父类的！**

### 一、 抽象基类：

只需要在模型的Meta类里添加`abstract=True`元数据项，就可以将一个模型转换为抽象基类。Django不会为这种类创建实际的数据库表，它们也没有管理器，不能被实例化也无法直接保存，它们就是用来被继承的。抽象基类完全就是用来保存子模型们共有的内容部分，达到重用的目的。当它们被继承时，它们的字段会全部复制到子模型中。看下面的例子：

```
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()
    
    class Meta:
        abstract = True
    
class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

Student模型将拥有name，age，home_group三个字段，并且CommonInfo模型不能当做一个正常的模型使用。

#### 抽象基类的Meta数据：

如果子类没有声明自己的Meta类，那么它将继承抽象基类的Meta类。下面的例子则扩展了基类的Meta：

```
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']
    
class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):
        db_table = 'student_info'
```

这里有几点要特别说明：

- 抽象基类中有的元数据，子模型没有的话，直接继承；
- 抽象基类中有的元数据，子模型也有的话，直接覆盖；
- 子模型可以额外添加元数据；
- 抽象基类中的`abstract=True`这个元数据不会被继承。也就是说如果想让一个抽象基类的子模型，同样成为一个抽象基类，那你必须显式的在该子模型的Meta中同样声明一个`abstract = True`；
- 有一些元数据对抽象基类无效，比如`db_table`，首先是抽象基类本身不会创建数据表，其次它的所有子类也不会按照这个元数据来设置表名。

#### 警惕related_name和related_query_name参数

如果在你的抽象基类中存在ForeignKey或者ManyToManyField字段，并且使用了`related_name`或者`related_query_name`参数，那么一定要小心了。因为按照默认规则，每一个子类都将拥有同样的字段，这显然会导致错误。为了解决这个问题，当你在抽象基类中使用`related_name`或者`related_query_name`参数时，它们两者的值中应该包含`%(app_label)s`和`%(class)s`部分：

- `%(class)s`用字段所属子类的小写名替换
- `%(app_label)s`用子类所属app的小写名替换

例如，对于`common/models.py`模块：

```
from django.db import models

class Base(models.Model):
    m2m = models.ManyToManyField(
    OtherModel,
    related_name="%(app_label)s_%(class)s_related",
    related_query_name="%(app_label)s_%(class)ss",
    )
    
    class Meta:
        abstract = True
        
class ChildA(Base):
    pass

class ChildB(Base):
    pass
```

对于另外一个应用中的`rare/models.py`:

```
from common.models import Base

class ChildB(Base):
    pass
```

对于上面的继承关系：

- `common.ChildA.m2m`字段的`reverse name`（反向关系名）应该是`common_childa_related`；`reverse query name`(反向查询名)应该是`common_childas`。
- `common.ChildB.m2m`字段的反向关系名应该是`common_childb_related`；反向查询名应该是`common_childbs`。
- `rare.ChildB.m2m`字段的反向关系名应该是`rare_childb_related`；反向查询名应该是`rare_childbs`。

当然，如果你不设置`related_name`或者`related_query_name`参数，这些问题就不存在了。

------

### 二、 多表继承

这种继承方式下，父类和子类都是独立自主、功能完整、可正常使用的模型，都有自己的数据库表，内部隐含了一个一对一的关系。例如：

```
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```

Restaurant将包含Place的所有字段，并且各有各的数据库表和字段，比如：

```
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```

如果一个Place对象同时也是一个Restaurant对象，你可以使用小写的子类名，在父类中访问它，例如：

```
>>> p = Place.objects.get(id=12)
# 如果p也是一个Restaurant对象，那么下面的调用可以获得该Restaurant对象。
>>> p.restaurant
<Restaurant: ...>
```

但是，如果这个Place是个纯粹的Place对象，并不是一个Restaurant对象，那么上面的调用方式会弹出`Restaurant.DoesNotExist`异常。

让我们看一组更具体的展示，注意里面的注释内容。

```
>>> from app1.models import Place, Restaurant  # 导入两个模型到shell里
>>> p1 = Place.objects.create(name='coff',address='address1')
>>> p1  # p1是个纯Place对象
<Place: Place object>
>>> p1.restaurant   # p1没有餐馆属性
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "C:\Python36\lib\site-packages\django\db\models\fields\related_descriptors.py", line 407, in __get__
    self.related.get_accessor_name()
django.db.models.fields.related_descriptors.RelatedObjectDoesNotExist: Place has no restaurant.
>>> r1 = Restaurant.objects.create(serves_hot_dogs=True,serves_pizza=False)
>>> r1  # r1在创建的时候，只赋予了2个字段的值
<Restaurant: Restaurant object>
>>> r1.place # 不能这么调用
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'Restaurant' object has no attribute 'place'
>>> r2 = Restaurant.objects.create(serves_hot_dogs=True,serves_pizza=False, name='pizza', address='address2')
>>> r2  # r2在创建时，提供了包括Place的字段在内的4个字段
<Restaurant: Restaurant object>
>>> r2.place   # 可以看出这么调用都是非法的，异想天开的
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'Restaurant' object has no attribute 'place'
>>> p2 = Place.objects.get(name='pizza') # 通过name，我们获取到了一个Place对象
>>> p2.restaurant  # 这个P2其实就是前面的r2
<Restaurant: Restaurant object>
>>> p2.restaurant.address
'address2'
>>> p2.restaurant.serves_hot_dogs
True
>>> lis = Place.objects.all()
>>> lis
<QuerySet [<Place: Place object>, <Place: Place object>, <Place: Place object>]>
>>> lis.values()
<QuerySet [{'id': 1, 'name': 'coff', 'address': 'address1'}, {'id': 2, 'name': '', 'address': ''}, {'id': 3, 'name': 'pizza', 'address': 'address2'}]>
>>> lis[2]
<Place: Place object>
>>> lis[2].serves_hot_dogs
Traceback (most recent call last):
  File "<console>", line 1, in <module>
AttributeError: 'Place' object has no attribute 'serves_hot_dogs'
>>> lis2 = Restaurant.objects.all()
>>> lis2
<QuerySet [<Restaurant: Restaurant object>, <Restaurant: Restaurant object>]>
>>> lis2.values()
<QuerySet [{'id': 2, 'name': '', 'address': '', 'place_ptr_id': 2, 'serves_hot_dogs': True, 'serves_pizza': False}, {'id': 3, 'name': 'pizza', 'address
': 'address2', 'place_ptr_id': 3, 'serves_hot_dogs': True, 'serves_pizza': False}]>
```

其机制内部隐含的OneToOne字段，形同下面所示：

```
place_ptr = models.OneToOneField(
    Place, on_delete=models.CASCADE,
    parent_link=True,
)
```

可以通过创建一个OneToOneField字段并设置 `parent_link=True`，自定义这个一对一字段。

------

#### Meta和多表继承

在多表继承的情况下，由于父类和子类都在数据库内有物理存在的表，父类的Meta类会对子类造成不确定的影响，因此，Django在这种情况下关闭了子类继承父类的Meta功能。这一点和抽象基类的继承方式有所不同。

但是，还有两个Meta元数据特殊一点，那就是`ordering`和`get_latest_by`，这两个参数是会被继承的。因此，如果在多表继承中，你不想让你的子类继承父类的上面两种参数，就必须在子类中显示的指出或重写。如下：

```
class ChildModel(ParentModel):
    # ...
    
    class Meta:
        # 移除父类对子类的排序影响
        ordering = []
```

------

#### 多表继承和反向关联

因为多表继承使用了一个隐含的OneToOneField来链接子类与父类，所以象上例那样，你可以从父类访问子类。但是这个OnetoOneField字段默认的`related_name`值与ForeignKey和 ManyToManyField默认的反向名称相同。如果你与父类或另一个子类做多对一或是多对多关系，你就必须在每个多对一和多对多字段上强制指定`related_name`。如果你没这么做，Django就会在你运行或验证(validation)时抛出异常。

仍以上面Place类为例，我们创建一个带有ManyToManyField字段的子类：

```
class Supplier(Place):
    customers = models.ManyToManyField(Place)
```

这会产生下面的错误：

```
Reverse query name for 'Supplier.customers' clashes with reverse query
name for 'Supplier.place_ptr'.
HINT: Add or change a related_name argument to the definition for
'Supplier.customers' or 'Supplier.place_ptr'.
```

解决方法是：向customers字段中添加`related_name`参数.

```
customers = models.ManyToManyField(Place, related_name='provider')。
```

------

### 三、 代理模型

使用多表继承时，父类的每个子类都会创建一张新数据表，通常情况下，这是我们想要的操作，因为子类需要一个空间来存储不包含在父类中的数据。但有时，你可能只想更改模型在Python层面的行为，比如更改默认的manager管理器，或者添加一个新方法。

代理模型就是为此而生的。你可以创建、删除、更新代理模型的实例，并且所有的数据都可以像使用原始模型（非代理类模型）一样被保存。不同之处在于你可以在代理模型中改变默认的排序方式和默认的manager管理器等等，而不会对原始模型产生影响。

**声明一个代理模型只需要将Meta中proxy的值设为True。**

例如你想给Person模型添加一个方法。你可以这样做：

```
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    
class MyPerson(Person):
    class Meta:
        proxy = True
        
    def do_something(self):
        # ...
        pass
```

MyPerson类将操作和Person类同一张数据库表。并且任何新的Person实例都可以通过MyPerson类进行访问，反之亦然。

```
>>> p = Person.objects.create(first_name="foobar")
>>> MyPerson.objects.get(first_name="foobar")
<MyPerson: foobar>
```

下面的例子通过代理进行排序，但父类却不排序：

```
class OrderedPerson(Person):
    class Meta:
        # 现在，普通的Person查询是无序的，而OrderedPerson查询会按照`last_name`排序。
        ordering = ["last_name"]
        proxy = True
```

**一些约束：**

- 代理模型必须继承自一个非抽象的基类，并且不能同时继承多个非抽象基类；
- 代理模型可以同时继承任意多个抽象基类，前提是这些抽象基类没有定义任何模型字段。
- 代理模型可以同时继承多个别的代理模型，前提是这些代理模型继承同一个非抽象基类。（早期Django版本不支持这一条）

**代理模型的管理器**

如不指定，则继承父类的管理器。如果你自己定义了管理器，那它就会成为默认管理器，但是父类的管理器依然有效。如下例子：

```
from django.db import models

class NewManager(models.Manager):
    # ...
    pass

class MyPerson(Person):
    objects = NewManager()

    class Meta:
        proxy = True
```

如果你想要向代理中添加新的管理器，而不是替换现有的默认管理器，你可以创建一个含有新的管理器的基类，并在继承时把他放在主基类的后面：

```
# Create an abstract class for the new manager.
class ExtraManagers(models.Model):
    secondary = NewManager()

    class Meta:
        abstract = True

class MyPerson(Person, ExtraManagers):
    class Meta:
        proxy = True
```

### 四、 多重继承

注意，多重继承和多表继承是两码事，两个概念。

Django的模型体系支持多重继承，就像Python一样。如果多个父类都含有Meta类，则只有第一个父类的会被使用，剩下的会忽略掉。

一般情况，能不要多重继承就不要，尽量让继承关系简单和直接，避免不必要的混乱和复杂。

**请注意**，继承同时含有相同id主键字段的类将抛出异常。为了解决这个问题，你可以在基类模型中显式的使用`AutoField`字段。如下例所示：

```
class Article(models.Model):
    article_id = models.AutoField(primary_key=True)
    ...

class Book(models.Model):
    book_id = models.AutoField(primary_key=True)
    ...

class BookReview(Book, Article):
    pass
```

或者使用一个共同的祖先来持有AutoField字段，并在直接的父类里通过一个OneToOne字段保持与祖先的关系，如下所示：

```
class Piece(models.Model):
    pass

class Article(Piece):
    article_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class Book(Piece):
    book_piece = models.OneToOneField(Piece, on_delete=models.CASCADE, parent_link=True)
    ...

class BookReview(Book, Article):
    pass
```

------

### 警告

在Python语言层面，子类可以拥有和父类相同的属性名，这样会造成覆盖现象。但是对于Django，如果继承的是一个非抽象基类，那么子类与父类之间不可以有相同的字段名！

比如下面是不行的！

```
class A(models.Model):
    name = models.CharField(max_length=30)

class B(A):
    name = models.CharField(max_length=30)
```

如果你执行`python manage.py makemigrations`会弹出下面的错误：

```
django.core.exceptions.FieldError: Local field 'name' in class 'B' clashes with field of the same name from base class 'A'.
```

但是！如果父类是个抽象基类就没有问题了(1.10版新增特性)，如下：

```
class A(models.Model):
    name = models.CharField(max_length=30)
    
    class Meta:
        abstract = True

class B(A):
    name = models.CharField(max_length=30)
```

## 六、用包来组织模型

在我们使用`python manage.py startapp xxx`命令创建新的应用时，Django会自动帮我们建立一个应用的基本文件组织结构，其中就包括一个`models.py`文件。通常，我们把当前应用的模型都编写在这个文件里，但是如果你的模型很多，那么将单独的`models.py`文件分割成一些独立的文件是个更好的做法。

首先，我们需要在应用中新建一个叫做`models`的包，再在包下创建一个`__init__.py`文件，这样才能确立包的身份。然后将`models.py`文件中的模型分割到一些`.py`文件中，比如`organic.py`和`synthetic.py`，然后删除`models.py`文件。最后在`__init__.py`文件中导入所有的模型。如下例所示：

```
#  myapp/models/__init__.py

from .organic import Person
from .synthetic import Robot
```

要显式明确地导入每一个模型，而不要使用`from .models import *`的方式，这样不会混淆命名空间，让代码更可读，更容易被分析工具使用。

## 七、查询操作

查询操作是Django的ORM框架中最重要的内容之一。我们建立模型、保存数据为的就是在需要的时候可以查询得到数据。Django自动为所有的模型提供了一套完善、方便、高效的API，一些重要的，我们要背下来，一些不常用的，要有印象，使用的时候可以快速查找参考手册。

------

本节的内容基于如下的一个博客应用模型：

```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```

### 一、创建对象

假设模型位于`mysite/blog/models.py`文件中，那么创建对象的方式如下：

```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

在后台，这会运行一条SQL的INSERT语句。如果你不显式地调用save()方法，Django不会立刻将该操作反映到数据库中。save()方法没有返回值，它可以接受一些额外的参数。

如果想要一行代码完成上面的操作，请使用`creat()`方法，它可以省略save的步骤：

```
b = Blog.objects.create(name='Beatles Blog', tagline='All the latest Beatles news.')
```

### 二、保存对象

使用save()方法，保存对数据库内已有对象的修改。例如如果已经存在b5对象在数据库内：

```
>>> b5.name = 'New name'
>>> b5.save()
```

在后台，这会运行一条SQL的UPDATE语句。如果你不显式地调用save()方法，Django不会立刻将该操作反映到数据库中。

#### 1. 保存外键和多对多字段

保存一个外键字段和保存普通字段没什么区别，只是要注意值的类型要正确。下面的例子，有一个Entry的实例entry和一个Blog的实例`cheese_blog`，然后把`cheese_blog`作为值赋给了entry的blog属性，最后调用save方法进行保存。

```
>>> from blog.models import Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

多对多字段的保存稍微有点区别，需要调用一个`add()`方法，而不是直接给属性赋值，但它不需要调用save方法。如下例所示：

```
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

在一行语句内，可以同时添加多个对象到多对多的字段，如下所示：

```
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

如果你指定或添加了错误类型的对象，Django会抛出异常。

### 三、检索对象

想要从数据库内检索对象，你需要基于模型类，通过管理器（Manager）构造一个查询结果集（QuerySet）。

每个QuerySet代表一些数据库对象的集合。它可以包含零个、一个或多个过滤器（filters）。Filters缩小查询结果的范围。在SQL语法中，一个QuerySet相当于一个SELECT语句，而filter则相当于WHERE或者LIMIT一类的子句。

通过模型的Manager获得QuerySet，每个模型至少具有一个Manager，默认情况下，它被称作`objects`，可以通过模型类直接调用它，但不能通过模型类的实例调用它，以此实现“表级别”操作和“记录级别”操作的强制分离。如下所示：

```
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
...
AttributeError: "Manager isn't accessible via Blog instances."
```

#### 1. 检索所有对象

使用`all()`方法，可以获取某张表的所有记录。

```
>>> all_entries = Entry.objects.all()
```

#### 2. 过滤对象

有两个方法可以用来过滤QuerySet的结果，分别是：

- `filter(**kwargs)`：返回一个根据指定参数查询出来的QuerySet
- `exclude(**kwargs)`：返回除了根据指定参数查询出来结果的QuerySet

其中，`**kwargs`参数的格式必须是Django设置的一些字段格式。

例如：

```
Entry.objects.filter(pub_date__year=2006)
```

它等同于：

```
Entry.objects.all().filter(pub_date__year=2006)
```

**链式过滤**

filter和exclude的结果依然是个QuerySet，因此它可以继续被filter和exclude，这就形成了链式过滤：

```
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime(2005, 1, 30)
... )
```

（这里需要注意的是，当在进行跨关系的链式过滤时，结果可能和你想象的不一样，参考下面的跨多值关系查询）

**被过滤的QuerySets都是唯一的**

每一次过滤，你都会获得一个全新的QuerySet，它和之前的QuerySet没有任何关系，可以完全独立的被保存，使用和重用。例如：

```
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

例子中的q2和q3虽然由q1得来，是q1的子集，但是都是独立自主存在的。同样q1也不会受到q2和q3的影响。

**QuerySets都是懒惰的**

一个创建QuerySets的动作不会立刻导致任何的数据库行为。你可以不断地进行filter动作一整天，Django不会运行任何实际的数据库查询动作，直到QuerySets被提交(evaluated)。

简而言之就是，只有碰到某些特定的操作，Django才会将所有的操作体现到数据库内，否则它们只是保存在内存和Django的层面中。这是一种提高数据库查询效率，减少操作次数的优化设计。看下面的例子：

```
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```

上面的例子，看起来执行了3次数据库访问，实际上只是在print语句时才执行1次访问。通常情况，QuerySets的检索不会立刻执行实际的数据库查询操作，直到出现类似print的请求，也就是所谓的evaluated。

#### 3. 检索单一对象

filter方法始终返回的是QuerySets，那怕只有一个对象符合过滤条件，返回的也是包含一个对象的QuerySets，这是一个集合类型对象，你可以简单的理解为Python列表，可迭代可循环可索引。

如果你确定你的检索只会获得一个对象，那么你可以使用Manager的get()方法来直接返回这个对象。

```
>>> one_entry = Entry.objects.get(pk=1)
```

在get方法中你可以使用任何filter方法中的查询参数，用法也是一模一样。

**注意**：使用get()方法和使用filter()方法然后通过[0]的方式分片，有着不同的地方。看似两者都是获取单一对象。但是，**如果在查询时没有匹配到对象，那么get()方法将抛出DoesNotExist异常**。这个异常是模型类的一个属性，在上面的例子中，如果不存在主键为1的Entry对象，那么Django将抛出`Entry.DoesNotExist`异常。

类似地，**在使用get()方法查询时，如果结果超过1个，则会抛出MultipleObjectsReturned异常**，这个异常也是模型类的一个属性。

**所以：get()方法要慎用！**

#### 4. 其它QuerySet方法

大多数情况下，需要从数据库中查找对象时，使用all()、 get()、filter() 和exclude()就行。针对QuerySet的方法还有很多，都是一些相对高级的用法。

#### 5. QuerySet使用限制

使用类似Python对列表进行切片的方法可以对QuerySet进行范围取值。它相当于SQL语句中的LIMIT和OFFSET子句。参考下面的例子：

```
>>> Entry.objects.all()[:5]      # 返回前5个对象
>>> Entry.objects.all()[5:10]    # 返回第6个到第10个对象
```

**注意：不支持负索引！例如 Entry.objects.all()[-1]是不允许的**

通常情况，切片操作会返回一个新的QuerySet，并且不会被立刻执行。但是有一个例外，那就是指定步长的时候，查询操作会立刻在数据库内执行，如下：

```
>>> Entry.objects.all()[:10:2]
```

若要获取单一的对象而不是一个列表（例如，SELECT foo FROM bar LIMIT 1），可以简单地使用索引而不是切片。例如，下面的语句返回数据库中根据标题排序后的第一条Entry：

```
>>> Entry.objects.order_by('headline')[0]
```

它相当于：

```
>>> Entry.objects.order_by('headline')[0:1].get()
```

注意：如果没有匹配到对象，那么第一种方法会抛出IndexError异常，而第二种方式会抛出DoesNotExist异常。

也就是说在使用get和切片的时候，要注意查询结果的元素个数。

#### 6. 字段查询

字段查询其实就是filter()、exclude()和get()等方法的关键字参数。
其基本格式是：`field__lookuptype=value`，**注意其中是双下划线**。
例如：

```
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
#　相当于：
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

其中的字段必须是模型中定义的字段之一。但是有一个例外，那就是ForeignKey字段，你可以为其添加一个“_id”后缀（单下划线）。这种情况下键值是外键模型的主键原生值。例如：

```
>>> Entry.objects.filter(blog_id=4)
```

如果你传递了一个非法的键值，查询函数会抛出TypeError异常。

Django的数据库API支持20多种查询类型，下面介绍一些常用的：

**exact：**

默认类型。如果你不提供查询类型，或者关键字参数不包含一个双下划线，那么查询类型就是这个默认的exact。

```
>>> Entry.objects.get(headline__exact="Cat bites dog")
# 相当于
# SELECT ... WHERE headline = 'Cat bites dog';
# 下面两个相当
>>> Blog.objects.get(id__exact=14)  # Explicit form
>>> Blog.objects.get(id=14)         # __exact is implied
```

**iexact：**

不区分大小写。

```
>>> Blog.objects.get(name__iexact="beatles blog")
# 匹配"Beatles Blog", "beatles blog",甚至"BeAtlES blOG".
```

**contains：**

表示包含的意思！大小写敏感！

```
Entry.objects.get(headline__contains='Lennon')
# 相当于
# SELECT ... WHERE headline LIKE '%Lennon%';
# 匹配'Today Lennon honored'，但不匹配'today lennon honored'
```

**icontains：**

contains的大小写不敏感模式。

**startswith和endswith**

以什么开头和以什么结尾。大小写敏感！

**istartswith和iendswith**

是不区分大小写的模式。

#### 7. 跨越关系查询

Django提供了强大并且直观的方式解决跨越关联的查询，它在后台自动执行包含JOIN的SQL语句。要跨越某个关联，只需使用关联的模型字段名称，并使用双下划线分隔，直至你想要的字段（可以链式跨越，无限跨度）。例如：

```
# 返回所有Blog的name为'Beatles Blog'的Entry对象
# 一定要注意，返回的是Entry对象，而不是Blog对象。
# objects前面用的是哪个class，返回的就是哪个class的对象。
>>> Entry.objects.filter(blog__name='Beatles Blog')
```

反之亦然，如果要引用一个反向关联，只需要使用模型的小写名!

```
# 获取所有的Blog对象，前提是它所关联的Entry的headline包含'Lennon'
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

如果你在多级关联中进行过滤而且其中某个中间模型没有满足过滤条件的值，Django将把它当做一个空的（所有的值都为NULL）但是合法的对象，不会抛出任何异常或错误。例如，在下面的过滤器中：

```
Blog.objects.filter(entry__authors__name='Lennon')
```

如果Entry中没有关联任何的author，那么它将当作其没有name，而不会因为没有author 引发一个错误。通常，这是比较符合逻辑的处理方式。唯一可能让你困惑的是当你使用`isnull`的时候：

```
Blog.objects.filter(entry__authors__name__isnull=True)
```

这将返回Blog对象，它关联的entry对象的author字段的name字段为空，以及Entry对象的author字段为空。如果你不需要后者，你可以这样写：

```
Blog.objects.filter(entry__authors__isnull=False,entry__authors__name__isnull=True)
```

**跨越多值的关系查询**

最基本的filter和exclude的关键字参数只有一个，这种情况很好理解。但是当关键字参数有多个，且是跨越外键或者多对多的情况下，那么就比较复杂，让人迷惑了。我们看下面的例子：

```
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)
```

这是一个跨外键、两个过滤参数的查询。此时我们理解两个参数之间属于-与“and”的关系，也就是说，过滤出来的BLog对象对应的entry对象必须同时满足上面两个条件。这点很好理解。也就是说**上面要求至少有一个entry同时满足两个条件**。

但是，看下面的用法：

```
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)
```

把两个参数拆开，放在两个filter调用里面，按照我们前面说过的链式过滤，这个结果应该和上面的例子一样。可实际上，它不一样，Django在这种情况下，将两个filter之间的关系设计为-或“or”，这真是让人头疼。

多对多关系下的多值查询和外键foreignkey的情况一样。

但是，更头疼的来了，exclude的策略设计的又和filter不一样！

```
Blog.objects.exclude(entry__headline__contains='Lennon',entry__pub_date__year=2008,)
```

这会排除headline中包含“Lennon”的Entry和在2008年发布的Entry，中间是一个-和“or”的关系！

那么要排除同时满足上面两个条件的对象，该怎么办呢？看下面：

```
Blog.objects.exclude(
entry=Entry.objects.filter(
    headline__contains='Lennon',
    pub_date__year=2008,
),
)
```

（有没有很坑爹的感觉？所以，建议在碰到跨关系的多值查询时，尽量使用Q查询）

#### 8. 使用F表达式引用模型的字段

到目前为止的例子中，我们都是将模型字段与常量进行比较。但是，如果你想将模型的一个字段与同一个模型的另外一个字段进行比较该怎么办？

使用Django提供的F表达式！

例如，为了查找comments数目多于pingbacks数目的Entry，可以构造一个`F()`对象来引用pingback数目，并在查询中使用该F()对象：

```
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```

Django支持对F()对象进行加、减、乘、除、取模以及幂运算等算术操作。两个操作数可以是常数和其它F()对象。例如查找comments数目比pingbacks两倍还要多的Entry，我们可以这么写：

```
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
```

为了查询rating比pingback和comment数目总和要小的Entry，我们可以这么写：

```
>>> Entry.objects.filter(rating__lt=F('n_comments') + F('n_pingbacks'))
```

你还可以在F()中使用双下划线来进行跨表查询。例如，查询author的名字与blog名字相同的Entry：

```
>>> Entry.objects.filter(authors__name=F('blog__name'))
```

对于date和date/time字段，还可以加或减去一个timedelta对象。下面的例子将返回发布时间超过3天后被修改的所有Entry：

```
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```

F()对象还支持`.bitand()`、`.bitor()`、`.bitrightshift()`和`.bitleftshift()`4种位操作，例如：

```
>>> F('somefield').bitand(16)
```

#### 9. 主键的快捷查询方式：pk

pk就是`primary key`的缩写。通常情况下，一个模型的主键为“id”，所以下面三个语句的效果一样：

```
>>> Blog.objects.get(id__exact=14) # Explicit form
>>> Blog.objects.get(id=14) # __exact is implied
>>> Blog.objects.get(pk=14) # pk implies id__exact
```

可以联合其他类型的参数：

```
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])
# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```

可以跨表操作：

```
>>> Entry.objects.filter(blog__id__exact=3) 
>>> Entry.objects.filter(blog__id=3) 
>>> Entry.objects.filter(blog__pk=3)
```

（**当主键不是id的时候，请注意了！**）

#### 10. 在LIKE语句中转义百分符号和下划线

在原生SQL语句中`%`符号有特殊的作用。Django帮你自动转义了百分符号和下划线，你可以和普通字符一样使用它们，如下所示：

```
>>> Entry.objects.filter(headline__contains='%')
# 它和下面的一样
# SELECT ... WHERE headline LIKE '%\%%';
```

#### 11. 缓存与查询集

每个QuerySet都包含一个缓存，用于减少对数据库的实际操作。理解这个概念，有助于你提高查询效率。

对于新创建的QuerySet，它的缓存是空的。当QuerySet第一次被提交后，数据库执行实际的查询操作，Django会把查询的结果保存在QuerySet的缓存内，随后的对于该QuerySet的提交将重用这个缓存的数据。

要想高效的利用查询结果，降低数据库负载，你必须善于利用缓存。看下面的例子，这会造成2次实际的数据库操作，加倍数据库的负载，同时由于时间差的问题，可能在两次操作之间数据被删除或修改或添加，导致脏数据的问题：

```
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```

为了避免上面的问题，好的使用方式如下，这只产生一次实际的查询操作，并且保持了数据的一致性：

```
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # 提交查询
>>> print([p.pub_date for p in queryset]) # 重用查询缓存
```

**何时不会被缓存**

有一些操作不会缓存QuerySet，例如切片和索引。这就导致这些操作没有缓存可用，每次都会执行实际的数据库查询操作。例如：

```
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # 查询数据库
>>> print(queryset[5]) # 再次查询数据库
```

但是，如果已经遍历过整个QuerySet，那么就相当于缓存过，后续的操作则会使用缓存，例如：

```
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # 查询数据库
>>> print(queryset[5]) # 使用缓存
>>> print(queryset[5]) # 使用缓存
```

下面的这些操作都将遍历QuerySet并建立缓存：

```
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

注意：简单的打印QuerySet并不会建立缓存，因为`__repr__()`调用只返回全部查询集的一个切片。

### 四、使用Q对象进行复杂查询

普通filter函数里的条件都是“and”逻辑，如果你想实现“or”逻辑怎么办？用Q查询！

Q来自`django.db.models.Q`，用于封装关键字参数的集合，可以作为关键字参数用于filter、exclude和get等函数。
例如：

```
from django.db.models import Q
Q(question__startswith='What')
```

可以使用“&”或者“|”或“~”来组合Q对象，分别表示与或非逻辑。它将返回一个新的Q对象。

```
Q(question__startswith='Who')|Q(question__startswith='What')
# 这相当于：
WHERE question LIKE 'Who%' OR question LIKE 'What%'
```

更多的例子：

```
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```

你也可以这么使用，默认情况下，以逗号分隔的都表示AND关系：

```
Poll.objects.get(
Q(question__startswith='Who'),
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
# 它相当于
# SELECT * from polls WHERE question LIKE 'Who%'
AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

当关键字参数和Q对象组合使用时，Q对象必须放在前面，如下例子：

```
Poll.objects.get(
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),question__startswith='Who',)
```

如果关键字参数放在Q对象的前面，则会报错。

### 五、比较对象

要比较两个模型实例，只需要使用python提供的双等号比较符就可以了。在后台，其实比较的是两个实例的主键的值。下面两种方法是等同的：

```
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
```

如果模型的主键不叫做“id”也没关系，后台总是会使用正确的主键名字进行比较，例如，如果一个模型的主键的名字是“name”，那么下面是相等的：

```
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
```

### 六、删除对象

删除对象使用的是对象的`delete()`方法。该方法将返回被删除对象的总数量和一个字典，字典包含了每种被删除对象的类型和该类型的数量。如下所示：

```
>>> e.delete()
(1, {'weblog.Entry': 1})
```

也可以批量删除。每个QuerySet都有一个delete()方法，它能删除该QuerySet的所有成员。例如：

```
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'webapp.Entry': 5})
```

需要注意的是，有可能不是每一个对象的delete方法都被执行。如果你改写了delete方法，为了确保对象被删除，你必须手动迭代QuerySet进行逐一删除操作。

当Django删除一个对象时，它默认使用SQL的ON DELETE CASCADE约束，也就是说，任何有外键指向要删除对象的对象将一起被删除。例如：

```
b = Blog.objects.get(pk=1)
# 下面的动作将删除该条Blog和所有的它关联的Entry对象
b.delete()
```

这种级联的行为可以通过的ForeignKey的`on_delete`参数自定义。

注意，`delete()`是唯一没有在管理器上暴露出来的方法。这是刻意设计的一个安全机制，用来防止你意外地请求类似`Entry.objects.delete()`的动作，而不慎删除了所有的条目。如果你确实想删除所有的对象，你必须明确地请求一个完全的查询集，像下面这样：

```
Entry.objects.all().delete()
```

### 七、复制模型实例

虽然没有内置的方法用于复制模型的实例，但还是很容易创建一个新的实例并将原实例的所有字段都拷贝过来。最简单的方法是将原实例的pk设置为None，这会创建一个新的实例copy。示例如下：

```
blog = Blog(name='My blog', tagline='Blogging is easy')
blog.save() # blog.pk == 1
#
blog.pk = None
blog.save() # blog.pk == 2
```

但是在使用继承的时候，情况会变得复杂，如果有下面一个Blog的子类：

```
class ThemeBlog(Blog):
    theme = models.CharField(max_length=200)

django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
django_blog.save() # django_blog.pk == 3
```

基于继承的工作机制，你必须同时将pk和id设为None：

```
django_blog.pk = None
django_blog.id = None
django_blog.save() # django_blog.pk == 4
```

对于外键和多对多关系，更需要进一步处理。例如，Entry有一个ManyToManyField到Author。 复制条目后，您必须为新条目设置多对多关系，像下面这样：

```
entry = Entry.objects.all()[0] # some previous entry
old_authors = entry.authors.all()
entry.pk = None
entry.save()
entry.authors.set(old_authors)
```

对于OneToOneField，还要复制相关对象并将其分配给新对象的字段，以避免违反一对一唯一约束。 例如，假设entry已经如上所述重复：

```
detail = EntryDetail.objects.all()[0]
detail.pk = None
detail.entry = entry
detail.save()
```

### 八、批量更新对象

使用`update()`方法可以批量为QuerySet中所有的对象进行更新操作。

```
# 更新所有2007年发布的entry的headline
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```

只可以对普通字段和ForeignKey字段使用这个方法。若要更新一个普通字段，只需提供一个新的常数值。若要更新ForeignKey字段，需设置新值为你想指向的新模型实例。例如：

```
>>> b = Blog.objects.get(pk=1)
# 修改所有的Entry，让他们都属于b
>>> Entry.objects.all().update(blog=b)
```

update方法会被立刻执行，并返回操作匹配到的行的数目（有可能不等于要更新的行的数量，因为有些行可能已经有这个新值了）。唯一的约束是：只能访问一张数据库表。你可以根据关系字段进行过滤，但你只能更新模型主表的字段。例如：

```
>>> b = Blog.objects.get(pk=1)
# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
```

要注意的是update()方法会直接转换成一个SQL语句，并立刻批量执行。它不会运行模型的save()方法，或者产生`pre_save`或`post_save`信号（调用`save()`方法产生）或者服从`auto_now`字段选项。如果你想保存QuerySet中的每个条目并确保每个实例的save()方法都被调用，你不需要使用任何特殊的函数来处理。只需要迭代它们并调用save()方法：

```
for item in my_queryset:
    item.save()
```

update方法可以配合F表达式。这对于批量更新同一模型中某个字段特别有用。例如增加Blog中每个Entry的pingback个数：

```
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
```

然而，与filter和exclude子句中的F()对象不同，在update中你不可以使用F()对象进行跨表操作，你只可以引用正在更新的模型的字段。如果你尝试使用F()对象引入另外一张表的字段，将抛出FieldError异常：

```
# THIS WILL RAISE A FieldError
>>> Entry.objects.update(headline=F('blog__name'))
```

### 九、关系的对象

利用本节一开始的模型，一个Entry对象e可以通过blog属性`e.blog`获取关联的Blog对象。反过来，Blog对象b可以通过`entry_set`属性`b.entry_set.all()`访问与它关联的所有Entry对象。

#### 1. 一对多（外键）

**正向查询:**

直接通过圆点加属性，访问外键对象：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog # 返回关联的Blog对象
```

要注意的是，对外键的修改，必须调用save方法进行保存，例如：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
```

如果一个外键字段设置有`null=True`属性，那么可以通过给该字段赋值为None的方法移除外键值：

```
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
```

在第一次对一个外键关系进行正向访问的时候，关系对象会被缓存。随后对同样外键关系对象的访问会使用这个缓存，例如：

```
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # 访问数据库，获取实际数据
>>> print(e.blog)  # 不会访问数据库，直接使用缓存的版本
```

请注意QuerySet的`select_related()`方法会递归地预填充所有的一对多关系到缓存中。例如：

```
>>> e = Entry.objects.select_related().get(id=2)
>>> print(e.blog)  # 不会访问数据库，直接使用缓存
>>> print(e.blog)  # 不会访问数据库，直接使用缓存
```

**反向查询:**

如果一个模型有ForeignKey，那么该ForeignKey所指向的外键模型的实例可以通过一个管理器进行反向查询，返回源模型的所有实例。默认情况下，这个管理器的名字为`FOO_set`，其中FOO是源模型的小写名称。该管理器返回的查询集可以用前面提到的方式进行过滤和操作。

```
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # Returns all Entry objects related to Blog.
# b.entry_set is a Manager that returns QuerySets.
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```

你可以在ForeignKey字段的定义中，通过设置`related_name`来重写`FOO_set`的名字。举例说明，如果你修改Entry模型`blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name=’entries’)`，那么上面的例子会变成下面的样子：

```
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() # Returns all Entry objects related to Blog.
# b.entries is a Manager that returns QuerySets.
>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
```

**使用自定义的反向管理器:**

默认情况下，用于反向关联的RelatedManager是该模型默认管理器的子类。如果你想为一个查询指定一个不同的管理器，你可以使用下面的语法：

```
from django.db import models

class Entry(models.Model):
    #...
    objects = models.Manager()  # 默认管理器
    entries = EntryManager()    # 自定义管理器

b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
```

当然，指定的自定义反向管理器也可以调用它的自定义方法：

```
b.entry_set(manager='entries').is_published()
```

**处理关联对象的其它方法:**

除了在前面定义的QuerySet方法之外，ForeignKey管理器还有其它方法用于处理关联的对象集合。下面是每个方法的概括。

add(obj1, obj2, ...)：添加指定的模型对象到关联的对象集中。

create(**kwargs)：创建一个新的对象，将它保存并放在关联的对象集中。返回新创建的对象。

remove(obj1, obj2, ...)：从关联的对象集中删除指定的模型对象。

clear()：清空关联的对象集。

set(objs)：重置关联的对象集。

若要一次性给关联的对象集赋值，使用set()方法，并给它赋值一个可迭代的对象集合或者一个主键值的列表。例如：

```
b = Blog.objects.get(id=1)
b.entry_set.set([e1, e2])
```

在这个例子中，e1和e2可以是完整的Entry实例，也可以是整数的主键值。

如果clear()方法可用，那么在将可迭代对象中的成员添加到集合中之前，将从`entry_set`中删除所有已经存在的对象。如果clear()方法不可用，那么将直接添加可迭代对象中的成员而不会删除所有已存在的对象。

这节中的每个反向操作都将立即在数据库内执行。所有的增加、创建和删除操作也将立刻自动地保存到数据库内。

#### 2. 多对多

多对多关系的两端都会自动获得访问另一端的API。这些API的工作方式与前面提到的“反向”一对多关系的用法一样。

唯一的区别在于属性的名称：定义ManyToManyField的模型使用该字段的属性名称，而“反向”模型使用源模型的小写名称加上'_set' （和一对多关系一样）。

```
e = Entry.objects.get(id=3)
e.authors.all() # Returns all Author objects for this Entry.
e.authors.count()
e.authors.filter(name__contains='John')
#
a = Author.objects.get(id=5)
a.entry_set.all() # Returns all Entry objects for this Author.
```

与外键字段中一样，在多对多的字段中也可以指定`related_name`名。

（注：在一个模型中，如果存在多个外键或多对多的关系指向同一个外部模型，必须给他们分别加上不同的`related_name`，用于反向查询）

#### 3. 一对一

一对一非常类似多对一关系，可以简单的通过模型的属性访问关联的模型。

```
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry, on_delete=models.CASCADE)
    details = models.TextField()

ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
```

不同之处在于反向查询的时候。一对一关系中的关联模型同样具有一个管理器对象，但是该管理器表示一个单一的对象而不是对象的集合：

```
e = Entry.objects.get(id=2)
e.entrydetail # 返回关联的EntryDetail对象
```

如果没有对象赋值给这个关系，Django将抛出一个DoesNotExist异常。
可以给反向关联进行赋值，方法和正向的关联一样：

```
e.entrydetail = ed
```

#### 4. 反向关联是如何实现的？

一些ORM框架需要你在关系的两端都进行定义。Django的开发者认为这违反了DRY (Don’t Repeat Yourself)原则，所以在Django中你只需要在一端进行定义。

那么这是怎么实现的呢？因为在关联的模型类没有被加载之前，一个模型类根本不知道有哪些类和它关联。

答案在`app registry`！在Django启动的时候，它会导入所有`INSTALLED_APPS`中的应用和每个应用中的模型模块。每创建一个新的模型时，Django会自动添加反向的关系到所有关联的模型。如果关联的模型还没有导入，Django将保存关联的记录并在关联的模型导入时添加这些关系。

由于这个原因，将模型所在的应用都定义在`INSTALLED_APPS`的应用列表中就显得特别重要。否则，反向关联将不能正确工作。

#### 5. 通过关联对象进行查询

涉及关联对象的查询与正常值的字段查询遵循同样的规则。当你指定查询需要匹配的值时，你可以使用一个对象实例或者对象的主键值。

例如，如果你有一个id=5的Blog对象b，下面的三个查询将是完全一样的：

```
Entry.objects.filter(blog=b) # 使用对象实例
Entry.objects.filter(blog=b.id) # 使用实例的id
Entry.objects.filter(blog=5) # 直接使用id
```

### 十、使用原生SQL语句

如果你发现需要编写的Django查询语句太复杂，你可以回归到手工编写SQL语句。Django对于编写原生的SQL查询有许多选项。

最后，需要注意的是Django的数据库层只是一个数据库接口。你可以利用其它的工具、编程语言或数据库框架来访问数据库，Django没有强制指定你非要使用它的某个功能或模块。

## 八、查询集API

本节将详细介绍查询集的API，它建立在下面的模型基础上，与上一节的模型相同：

```
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):              # __unicode__ on Python 2
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```

### 一、QuerySet何时被提交

在内部，创建、过滤、切片和传递一个QuerySet不会真实操作数据库，在你对查询集提交之前，不会发生任何实际的数据库操作。

可以使用下列方法对QuerySet提交查询操作：

- **迭代**

QuerySet是可迭代的，在首次迭代查询集时执行实际的数据库查询。 例如， 下面的语句会将数据库中所有Entry的headline打印出来：

```
for e in Entry.objects.all():
    print(e.headline)
```

- **切片**：如果使用切片的”step“参数，Django 将执行数据库查询并返回一个列表。
- **Pickling/缓存**
- **repr()**
- **len()**：当你对QuerySet调用len()时， 将提交数据库操作。
- **list()**：对QuerySet调用list()将强制提交操作`entry_list = list(Entry.objects.all())`
- **bool()**

测试布尔值，像这样：

```
if Entry.objects.filter(headline="Test"):
   print("There is at least one Entry with the headline Test")
```

注：如果你需要知道是否存在至少一条记录（而不需要真实的对象），使用exists() 将更加高效。

### 二、QuerySet

下面是对于QuerySet的正式定义：

```
class QuerySet(model=None, query=None, using=None)[source]
```

QuerySet类具有两个公有属性用于内省：

ordered：如果QuerySet是排好序的则为True，否则为False。

db：如果现在执行，则返回使用的数据库。

### 三、返回新QuerySets的API

**以下的方法都将返回一个新的QuerySets。**重点是加粗的几个API，其它的使用场景很少。

| 方法名                | 解释                                         |
| --------------------- | -------------------------------------------- |
| **filter()**          | 过滤查询对象。                               |
| **exclude()**         | 排除满足条件的对象                           |
| **annotate()**        | 使用聚合函数                                 |
| **order_by()**        | 对查询集进行排序                             |
| **reverse()**         | 反向排序                                     |
| **distinct()**        | 对查询集去重                                 |
| **values()**          | 返回包含对象具体值的字典的QuerySet           |
| **values_list()**     | 与values()类似，只是返回的是元组而不是字典。 |
| dates()               | 根据日期获取查询集                           |
| datetimes()           | 根据时间获取查询集                           |
| **none()**            | 创建空的查询集                               |
| **all()**             | 获取所有的对象                               |
| union()               | 并集                                         |
| intersection()        | 交集                                         |
| difference()          | 差集                                         |
| **select_related()**  | 附带查询关联对象                             |
| `prefetch_related()`  | 预先查询                                     |
| extra()               | 附加SQL查询                                  |
| defer()               | 不加载指定字段                               |
| only()                | 只加载指定的字段                             |
| using()               | 选择数据库                                   |
| `select_for_update()` | 锁住选择的对象，直到事务结束。               |
| raw()                 | 接收一个原始的SQL查询                        |

#### 1. filter()

filter(**kwargs)

返回满足查询参数的对象集合。

查找的参数（**kwargs）应该满足下文字段查找中的格式。多个参数之间是和AND的关系。

#### 2. exclude()

exclude(**kwargs)

返回一个新的QuerySet，它包含**不**满足给定的查找参数的对象。

查找的参数（**kwargs）应该满足下文字段查找中的格式。多个参数通过AND连接，然后所有的内容放入NOT() 中。

下面的示例**排除**所有`pub_date`晚于2005-1-3**且**headline为“Hello” 的记录：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3), headline='Hello')
```

下面的示例**排除**所有`pub_date`晚于2005-1-3**或者**headline 为“Hello” 的记录：

```
Entry.objects.exclude(pub_date__gt=datetime.date(2005, 1, 3)).exclude(headline='Hello')
```

#### 3. annotate()

annotate(*args, **kwargs)

使用提供的聚合表达式查询对象。

表达式可以是简单的值、对模型（或任何关联模型）上的字段的引用或者聚合表达式（平均值、总和等）。

annotate()的每个参数都是一个annotation，它将添加到返回的QuerySet每个对象中。

关键字参数指定的Annotation将使用关键字作为Annotation 的别名。 匿名参数的别名将基于聚合函数的名称和模型的字段生成。 只有引用单个字段的聚合表达式才可以使用匿名参数。 其它所有形式都必须用关键字参数。

例如，如果正在操作一个Blog列表，你可能想知道每个Blog有多少Entry：

```
>>> from django.db.models import Count
>>> q = Blog.objects.annotate(Count('entry'))
# The name of the first blog
>>> q[0].name
'Blogasaurus'
# The number of entries on the first blog
>>> q[0].entry__count
42
```

Blog模型本身没有定义`entry__count`属性，但是通过使用一个关键字参数来指定聚合函数，可以控制Annotation的名称：

```
>>> q = Blog.objects.annotate(number_of_entries=Count('entry'))
# The number of entries on the first blog, using the name provided
>>> q[0].number_of_entries
42
```

#### 4. order_by()

order_by(*fields)

默认情况下，根据模型的Meta类中的ordering属性对QuerySet中的对象进行排序

```
Entry.objects.filter(pub_date__year=2005).order_by('-pub_date', 'headline')
```

上面的结果将按照`pub_date`降序排序，然后再按照headline升序排序。"-pub_date"前面的负号表示降序顺序。 升序是默认的。 要随机排序，使用"?"，如下所示：

```
Entry.objects.order_by('?')
```

注：`order_by('?')`可能耗费资源且很慢，这取决于使用的数据库。

若要按照另外一个模型中的字段排序，可以使用查询关联模型的语法。即通过字段的名称后面跟两个下划线（`__`），再加上新模型中的字段的名称，直到希望连接的模型。 像这样：

```
Entry.objects.order_by('blog__name', 'headline')
```

如果排序的字段与另外一个模型关联，Django将使用关联的模型的默认排序，或者如果没有指定Meta.ordering将通过关联的模型的主键排序。 例如，因为Blog模型没有指定默认的排序：

```
Entry.objects.order_by('blog')
```

与以下相同：

```
Entry.objects.order_by('blog__id')
```

如果Blog设置了`ordering = ['name']`，那么第一个QuerySet将等同于：

```
Entry.objects.order_by('blog__name')
```

还可以通过调用表达式的desc()或者asc()方法：

```
Entry.objects.order_by(Coalesce('summary', 'headline').desc())
```

考虑下面的情况，指定一个多值字段来排序（例如，一个ManyToManyField 字段或者ForeignKey 字段的反向关联）：

```
class Event(Model):
   parent = models.ForeignKey(
       'self',
       on_delete=models.CASCADE,
       related_name='children',
   )
   date = models.DateField()

Event.objects.order_by('children__date')
```

在这里，每个Event可能有多个排序数据；具有多个children的每个Event将被多次返回到`order_by()`创建的新的QuerySet中。 换句话说，用`order_by()`方法对QuerySet对象进行操作会返回一个扩大版的新QuerySet对象。因此，使用多值字段对结果进行排序时要格外小心。

没有方法指定排序是否考虑大小写。 对于大小写的敏感性，Django将根据数据库中的排序方式排序结果。

可以通过Lower将一个字段转换为小写来排序，它将达到大小写一致的排序：

```
Entry.objects.order_by(Lower('headline').desc())
```

可以通过检查`QuerySet.ordered`属性来知道查询是否是排序的。

每个`order_by()`都将清除前面的任何排序。 例如下面的查询将按照`pub_date`排序，而不是headline：

```
Entry.objects.order_by('headline').order_by('pub_date')
```

#### 5. reverse()

reverse()

反向排序QuerySet中返回的元素。 第二次调用reverse()将恢复到原有的排序。

如要获取QuerySet中最后五个元素，可以这样做：

```
my_queryset.reverse()[:5]
```

这与Python直接使用负索引有点不一样。 Django不支持负索引，只能曲线救国。

#### 6. distinct()

distinct(*fields)

去除查询结果中重复的行。

默认情况下，QuerySet不会去除重复的行。当查询跨越多张表的数据时，QuerySet可能得到重复的结果，这时候可以使用distinct()进行去重。

#### 7. values()

values(*fields, **expressions)

返回一个包含数据的字典的queryset，而不是模型实例。

每个字典表示一个对象，键对应于模型对象的属性名称。

下面的例子将values() 与普通的模型对象进行比较：

```
# 列表中包含的是Blog对象
>>> Blog.objects.filter(name__startswith='Beatles')
<QuerySet [<Blog: Beatles Blog>]>
# 列表中包含的是数据字典
>>> Blog.objects.filter(name__startswith='Beatles').values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
```

该方法接收可选的位置参数`*fields`，它指定values()应该限制哪些字段。如果指定字段，每个字典将只包含指定的字段的键/值。如果没有指定字段，每个字典将包含数据库表中所有字段的键和值。

例如：

```
>>> Blog.objects.values()
<QuerySet [{'id': 1, 'name': 'Beatles Blog', 'tagline': 'All the latest Beatles news.'}]>
>>> Blog.objects.values('id', 'name')
<QuerySet [{'id': 1, 'name': 'Beatles Blog'}]>
```

values()方法还有关键字参数`**expressions`，这些参数将传递给`annotate()`：

```
>>> from django.db.models.functions import Lower
>>> Blog.objects.values(lower_name=Lower('name'))
<QuerySet [{'lower_name': 'beatles blog'}]>
```

在values()子句中的聚合应用于相同values()子句中的其他参数之前。 如果需要按另一个值分组，请将其添加到较早的values()子句中。 像这样：

```
>>> from django.db.models import Count
>>> Blog.objects.values('author', entries=Count('entry'))
<QuerySet [{'author': 1, 'entries': 20}, {'author': 1, 'entries': 13}]>
>>> Blog.objects.values('author').annotate(entries=Count('entry'))
<QuerySet [{'author': 1, 'entries': 33}]>
```

注意：

如果你有一个字段foo是一个ForeignKey，默认的`foo_id`参数返回的字典中将有一个叫做foo 的键，因为这是保存实际值的那个隐藏的模型属性的名称。 当调用`foo_id`并传递字段的名称，传递foo 或values()都可以，得到的结果是相同的。像这样：

```
>>> Entry.objects.values()
<QuerySet [{'blog_id': 1, 'headline': 'First Entry', ...}, ...]>
>>> Entry.objects.values('blog')
<QuerySet [{'blog': 1}, ...]>
>>> Entry.objects.values('blog_id')
<QuerySet [{'blog_id': 1}, ...]>
```

当values()与distinct()一起使用时，注意排序可能影响最终的结果。

如果values()子句位于extra()调用之后，extra()中的select参数定义的字段必须显式包含在values()调用中。 values( 调用后面的extra( 调用将忽略选择的额外的字段。

在values()之后调用only()和defer()不太合理，所以将引发一个NotImplementedError。

可以通过ManyToManyField、ForeignKey 和 OneToOneFiel 属性反向引用关联的模型的字段：

```
>>> Blog.objects.values('name', 'entry__headline')
<QuerySet [{'name': 'My blog', 'entry__headline': 'An entry'},
     {'name': 'My blog', 'entry__headline': 'Another entry'}, ...]>
```

#### 8. values_list()

values_list(*fields, flat=False)

与values()类似，只是在迭代时返回的是元组而不是字典。每个元组包含传递给`values_list()`调用的相应字段或表达式的值，因此第一个项目是第一个字段等。 像这样：

```
>>> Entry.objects.values_list('id', 'headline')
<QuerySet [(1, 'First entry'), ...]>
>>> from django.db.models.functions import Lower
>>> Entry.objects.values_list('id', Lower('headline'))
<QuerySet [(1, 'first entry'), ...]>
```

如果只传递一个字段，还可以传递flat参数。 如果为True，它表示返回的结果为单个值而不是元组。 如下所示：

```
>>> Entry.objects.values_list('id').order_by('id')
<QuerySet[(1,), (2,), (3,), ...]>
>>> Entry.objects.values_list('id', flat=True).order_by('id')
<QuerySet [1, 2, 3, ...]>
```

如果有多个字段，传递flat将发生错误。

如果不传递任何值给`values_list()`，它将返回模型中的所有字段，以在模型中定义的顺序。

常见的情况是获取某个模型实例的特定字段值。可以使用`values_list()`，然后调用get()：

```
>>> Entry.objects.values_list('headline', flat=True).get(pk=1)
'First entry'
```

`values()`和`values_list()`都用于特定情况下的优化：检索数据子集，而无需创建模型实例。

注意通过ManyToManyField进行查询时的行为：

```
>>> Author.objects.values_list('name', 'entry__headline')
<QuerySet [('Noam Chomsky', 'Impressions of Gaza'),
 ('George Orwell', 'Why Socialists Do Not Believe in Fun'),
 ('George Orwell', 'In Defence of English Cooking'),
 ('Don Quixote', None)]>
```

类似地，当查询反向外键时，对于没有任何作者的条目，返回None。

```
>>> Entry.objects.values_list('authors')
<QuerySet [('Noam Chomsky',), ('George Orwell',), (None,)]>
```

#### 9. dates()

dates(field, kind, order='ASC')

返回一个QuerySet，表示QuerySet内容中特定类型的所有可用日期的`datetime.date`对象列表。

field参数是模型的DateField的名称。 kind参数应为"year"，"month"或"day"。 结果列表中的每个datetime.date对象被截取为给定的类型。

"year" 返回对应该field的所有不同年份值的列表。

"month"返回字段的所有不同年/月值的列表。

"day"返回字段的所有不同年/月/日值的列表。

order参数默认为'ASC'，或者'DESC'。 它指定如何排序结果。

例子：

```
>>> Entry.objects.dates('pub_date', 'year')
[datetime.date(2005, 1, 1)]
>>> Entry.objects.dates('pub_date', 'month')
[datetime.date(2005, 2, 1), datetime.date(2005, 3, 1)]
>>> Entry.objects.dates('pub_date', 'day')
[datetime.date(2005, 2, 20), datetime.date(2005, 3, 20)]
>>> Entry.objects.dates('pub_date', 'day', order='DESC')
[datetime.date(2005, 3, 20), datetime.date(2005, 2, 20)]
>>> Entry.objects.filter(headline__contains='Lennon').dates('pub_date', 'day')
[datetime.date(2005, 3, 20)]
```

#### 10. datetimes()

datetimes(field_name, kind, order='ASC', tzinfo=None)

返回QuerySet，为datetime.datetime对象的列表，表示QuerySet内容中特定种类的所有可用日期。

`field_name`应为模型的DateTimeField的名称。

kind参数应为"hour"，"minute"，"month"，"year"，"second"或"day"。

结果列表中的每个datetime.datetime对象被截取到给定的类型。

order参数默认为'ASC'，或者'DESC'。 它指定如何排序结果。

tzinfo参数定义在截取之前将数据时间转换到的时区。

#### 11. none()

none()

调用none()将创建一个不返回任何对象的查询集，并且在访问结果时不会执行任何查询。

例子：

```
>>> Entry.objects.none()
<QuerySet []>
>>> from django.db.models.query import EmptyQuerySet
>>> isinstance(Entry.objects.none(), EmptyQuerySet)
True
```

#### 12. all()

all()

返回当前QuerySet（或QuerySet子类）的副本。通常用于获取全部QuerySet对象。

#### 13. union()

union(*other_qs, all=False)

Django中的新功能1.11。也就是集合中并集的概念！

使用SQL的UNION运算符组合两个或更多个QuerySet的结果。例如：

```
>>> qs1.union(qs2, qs3)
```

默认情况下，UNION操作符仅选择不同的值。 要允许重复值，请使用all=True参数。

#### 14. intersection()

intersection(*other_qs)

Django中的新功能1.11。也就是集合中交集的概念！

使用SQL的INTERSECT运算符返回两个或更多个QuerySet的共有元素。例如：

```
>>> qs1.intersection(qs2, qs3)
```

#### 15. difference()

difference(*other_qs)

Django中的新功能1.11。也就是集合中差集的概念！

使用SQL的EXCEPT运算符只保留QuerySet中的元素，但不保留其他QuerySet中的元素。例如：

```
>>> qs1.difference(qs2, qs3)
```

#### 16. select_related()

select_related(*fields)

沿着外键关系查询关联的对象的数据。这会生成一个复杂的查询并引起性能的损耗，但是在以后使用外键关系时将不需要再次数据库查询。

下面的例子解释了普通查询和`select_related()`查询的区别。 下面是一个标准的查询：

```
# 访问数据库。
e = Entry.objects.get(id=5)
# 再次访问数据库以得到关联的Blog对象。
b = e.blog
```

下面是一个`select_related`查询：

```
# 访问数据库。
e = Entry.objects.select_related('blog').get(id=5)
# 不会访问数据库，因为e.blog已经在前面的查询中获得了。
b = e.blog
```

`select_related()`可用于objects任何的查询集：

```
from django.utils import timezone

# Find all the blogs with entries scheduled to be published in the future.
blogs = set()

for e in Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog'):
    # 没有select_related()，下面的语句将为每次循环迭代生成一个数据库查询,以获得每个entry关联的blog。
    blogs.add(e.blog)
```

`filter()`和`select_related()`的顺序不重要。 下面的查询集是等同的：

```
Entry.objects.filter(pub_date__gt=timezone.now()).select_related('blog')
Entry.objects.select_related('blog').filter(pub_date__gt=timezone.now())
```

可以沿着外键查询。 如果有以下模型：

```
from django.db import models

class City(models.Model):
    # ...
    pass

class Person(models.Model):
    # ...
    hometown = models.ForeignKey(
        City,
        on_delete=models.SET_NULL,
        blank=True,
        null=True,
    )

class Book(models.Model):
    # ...
    author = models.ForeignKey(Person, on_delete=models.CASCADE)
```

调用`Book.objects.select_related('author__hometown').get(id=4)`将缓存相关的Person 和相关的City：

```
b = Book.objects.select_related('author__hometown').get(id=4)
p = b.author         # Doesn't hit the database.
c = p.hometown       # Doesn't hit the database.
b = Book.objects.get(id=4) # No select_related() in this example.
p = b.author         # Hits the database.
c = p.hometown       # Hits the database.
```

在传递给`select_related()`的字段中，可以使用任何ForeignKey和OneToOneField。

在传递给`select_related`的字段中，还可以反向引用OneToOneField。也就是说，可以回溯到定义OneToOneField 的字段。 此时，可以使用关联对象字段的`related_name`，而不要指定字段的名称。

#### 17. prefetch_related()

prefetch_related(*lookups)

在单个批处理中自动检索每个指定查找的相关对象。

与`select_related`类似，但是策略是完全不同的。

假设有这些模型：

```
from django.db import models

class Topping(models.Model):
    name = models.CharField(max_length=30)

class Pizza(models.Model):
    name = models.CharField(max_length=50)
    toppings = models.ManyToManyField(Topping)

    def __str__(self):              # __unicode__ on Python 2
        return "%s (%s)" % (
            self.name,
            ", ".join(topping.name for topping in self.toppings.all()),
        )
```

并运行：

```
>>> Pizza.objects.all()
["Hawaiian (ham, pineapple)", "Seafood (prawns, smoked salmon)"...
```

问题是每次QuerySet要求`Pizza.objects.all()`查询数据库，因此`self.toppings.all()`将在`Pizza Pizza.__str__()`中的每个项目的Toppings表上运行查询。

可以使用`prefetch_related`减少为只有两个查询：

```
>>> Pizza.objects.all().prefetch_related('toppings')
```

这意味着现在每次`self.toppings.all()`被调用，不会再去数据库查找，而是在一个预取的QuerySet缓存中查找。

还可以使用正常连接语法来执行相关字段的相关字段。 假设在上面的例子中增加一个额外的模型：

```
class Restaurant(models.Model):
    pizzas = models.ManyToManyField(Pizza, related_name='restaurants')
    best_pizza = models.ForeignKey(Pizza, related_name='championed_by')
```

以下是合法的：

```
>>> Restaurant.objects.prefetch_related('pizzas__toppings')
```

这将预取所有属于餐厅的比萨饼，和所有属于那些比萨饼的配料。 这将导致总共3个查询 - 一个用于餐馆，一个用于比萨饼，一个用于配料。

```
>>> Restaurant.objects.prefetch_related('best_pizza__toppings')
```

这将获取最好的比萨饼和每个餐厅最好的披萨的所有配料。 这将在3个表中查询 - 一个为餐厅，一个为“最佳比萨饼”，一个为一个为配料。

当然，也可以使用`best_pizza`来获取`select_related`关系，以将查询数减少为2：

```
>>> Restaurant.objects.select_related('best_pizza').prefetch_related('best_pizza__toppings')
```

#### 18. extra()

extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)

有些情况下，Django的查询语法难以简单的表达复杂的WHERE子句，对于这种情况,可以在extra()生成的SQL从句中注入新子句。使用这种方法作为最后的手段，这是一个旧的API，在将来的某个时候可能被弃用。仅当无法使用其他查询方法表达查询时才使用它。

例如：

```
>>> qs.extra(
...     select={'val': "select col from sometable where othercol = %s"},
...     select_params=(someparam,),
... )
```

相当于：

```
>>> qs.annotate(val=RawSQL("select col from sometable where othercol = %s", (someparam,)))
```

#### 19. defer()

defer(*fields)

在一些复杂的数据建模情况下，模型可能包含大量字段，其中一些可能包含大尺寸数据（例如文本字段），将它们转换为Python对象需要花费很大的代价。

当最初获取数据时不知道是否需要这些特定字段的情况下，如果正在使用查询集的结果，可以告诉Django不要从数据库中检索它们。

通过传递字段名称到defer()实现不加载：

```
Entry.objects.defer("headline", "body")
```

具有延迟加载字段的查询集仍将返回模型实例。

每个延迟字段将在你访问该字段时从数据库中检索（每次只检索一个，而不是一次检索所有的延迟字段）。

可以多次调用defer()。 每个调用都向延迟集添加新字段：

```
# 延迟body和headline两个字段。
Entry.objects.defer("body").filter(rating=5).defer("headline")
```

字段添加到延迟集的顺序无关紧要。对已经延迟的字段名称再次defer()没有问题（该字段仍将被延迟）。

可以使用标准的双下划线符号来分隔关联的字段，从而加载关联模型中的字段：

```
Blog.objects.select_related().defer("entry__headline", "entry__body")
```

如果要清除延迟字段集，将None作为参数传递到defer()：

```
# 立即加载所有的字段。
my_queryset.defer(None)
```

defer()方法（及其兄弟，only()）仅适用于高级用例，它们提供了数据加载的优化方法。

#### 20. only()

only(*fields)

only()方法与defer()相反。

如果有一个模型几乎所有的字段需要延迟，使用only()指定补充的字段集可以使代码更简单。

假设有一个包含字段biography、age和name的模型。 以下两个查询集是相同的，就延迟字段而言：

```
Person.objects.defer("age", "biography")
Person.objects.only("name")
```

每当你调用only()时，它将替换立即加载的字段集。因此，对only()的连续调用的结果是只有最后一次调用的字段被考虑：

```
# This will defer all fields except the headline.
Entry.objects.only("body", "rating").only("headline")
```

由于defer()以递增方式动作（向延迟列表中添加字段），因此你可以结合only()和defer()调用：

```
# Final result is that everything except "headline" is deferred.
Entry.objects.only("headline", "body").defer("body")
# Final result loads headline and body immediately (only() replaces any
# existing set of fields).
Entry.objects.defer("body").only("headline", "body")
```

当对具有延迟字段的实例调用save()时，仅保存加载的字段。

#### 21. using()

using(alias)

如果正在使用多个数据库，这个方法用于指定在哪个数据库上查询QuerySet。方法的唯一参数是数据库的别名，定义在DATABASES。

例如：

```
# queries the database with the 'default' alias.
>>> Entry.objects.all()
# queries the database with the 'backup' alias
>>> Entry.objects.using('backup')
```

#### 22. select_for_update()

select_for_update(nowait=False, skip_locked=False)

返回一个锁住行直到事务结束的查询集，如果数据库支持，它将生成一个`SELECT ... FOR UPDATE`语句。

例如：

```
entries = Entry.objects.select_for_update().filter(author=request.user)
```

所有匹配的行将被锁定，直到事务结束。这意味着可以通过锁防止数据被其它事务修改。

一般情况下如果其他事务锁定了相关行，那么本查询将被阻塞，直到锁被释放。使用`select_for_update(nowait=True)`将使查询不阻塞。如果其它事务持有冲突的锁,那么查询将引发`DatabaseError`异常。也可以使用`select_for_update(skip_locked=True)`忽略锁定的行。nowait和`skip_locked`是互斥的。

目前，postgresql，oracle和mysql数据库后端支持`select_for_update()`。但是，MySQL不支持nowait和`skip_locked`参数。

#### 23. raw()

raw(raw_query, params=None, translations=None)

接收一个原始的SQL查询，执行它并返回一个`django.db.models.query.RawQuerySet`实例。

这个RawQuerySet实例可以迭代，就像普通的QuerySet一样。

## 九、Django中不返回QuerySets的API

以下的方法不会返回QuerySets，但是作用非常强大，尤其是粗体显示的方法，需要背下来。

| 方法名                 | 解释                             |
| ---------------------- | -------------------------------- |
| **get()**              | 获取单个对象                     |
| **create()**           | 创建对象，无需save()             |
| **get_or_create()**    | 查询对象，如果没有找到就新建对象 |
| **update_or_create()** | 更新对象，如果没有找到就创建对象 |
| `bulk_create()`        | 批量创建对象                     |
| **count()**            | 统计对象的个数                   |
| `in_bulk()`            | 根据主键值的列表，批量返回对象   |
| `iterator()`           | 获取包含对象的迭代器             |
| **latest()**           | 获取最近的对象                   |
| **earliest()**         | 获取最早的对象                   |
| **first()**            | 获取第一个对象                   |
| **last()**             | 获取最后一个对象                 |
| **aggregate()**        | 聚合操作                         |
| **exists()**           | 判断queryset中是否有对象         |
| **update()**           | 批量更新对象                     |
| **delete()**           | 批量删除对象                     |
| as_manager()           | 获取管理器                       |

### 1. get()

get(**kwargs)

返回按照查询参数匹配到的单个对象，参数的格式应该符合Field lookups的要求。

如果匹配到的对象个数不只一个的话，触发MultipleObjectsReturned异常

如果根据给出的参数匹配不到对象的话，触发DoesNotExist异常。例如：

```
Entry.objects.get(id='foo') # raises Entry.DoesNotExist
```

DoesNotExist异常从`django.core.exceptions.ObjectDoesNotExist`继承，可以定位多个DoesNotExist异常。 例如：

```
from django.core.exceptions import ObjectDoesNotExist
try:
    e = Entry.objects.get(id=3)
    b = Blog.objects.get(id=1)
except ObjectDoesNotExist:
    print("Either the entry or blog doesn't exist.")
```

如果希望查询器只返回一行，则可以使用get()而不使用任何参数来返回该行的对象：

```
entry = Entry.objects.filter(...).exclude(...).get()
```

### 2. create()

create(**kwargs)

在一步操作中同时创建并且保存对象的便捷方法.

```
p = Person.objects.create(first_name="Bruce", last_name="Springsteen")
```

等于:

```
p = Person(first_name="Bruce", last_name="Springsteen")
p.save(force_insert=True)
```

参数`force_insert`表示强制创建对象。如果model中有一个你手动设置的主键，并且这个值已经存在于数据库中, 调用create()将会失败并且触发IntegrityError因为主键必须是唯一的。如果你手动设置了主键，做好异常处理的准备。

### 3. get_or_create()

get_or_create(defaults=None, **kwargs)

**通过kwargs来查询对象的便捷方法（如果模型中的所有字段都有默认值，可以为空），如果该对象不存在则创建一个新对象**。

该方法**返回一个由(object, created)组成的元组**，元组中的object 是一个查询到的或者是被创建的对象， created是一个表示是否创建了新的对象的布尔值。

对于下面的代码：

```
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
except Person.DoesNotExist:
    obj = Person(first_name='John', last_name='Lennon', birthday=date(1940, 10, 9))
    obj.save()
```

如果模型的字段数量较大的话，这种模式就变的非常不易用了。 上面的示例可以用`get_or_create()`重写 :

```
obj, created = Person.objects.get_or_create(
    first_name='John',
    last_name='Lennon',
    defaults={'birthday': date(1940, 10, 9)},
)
```

任何传递给`get_or_create()`的关键字参数，除了一个可选的defaults，都将传递给get()调用。 如果查找到一个对象，返回一个包含匹配到的对象以及False 组成的元组。 如果查找到的对象超过一个以上，将引发MultipleObjectsReturned。如果查找不到对象，`get_or_create()`将会实例化并保存一个新的对象，返回一个由新的对象以及True组成的元组。新的对象将会按照以下的逻辑创建:

```
params = {k: v for k, v in kwargs.items() if '__' not in k}
params.update({k: v() if callable(v) else v for k, v in defaults.items()})
obj = self.model(**params)
obj.save()
```

它表示从非'defaults' 且不包含双下划线的关键字参数开始。然后将defaults的内容添加进来，覆盖必要的键，并使用结果作为关键字参数传递给模型类。

如果有一个名为`defaults__exact`的字段，并且想在`get_or_create()`时用它作为精确查询，只需要使用defaults，像这样：

```
Foo.objects.get_or_create(defaults__exact='bar', defaults={'defaults': 'baz'})
```

当你使用手动指定的主键时，`get_or_create()`方法与`create()`方法有相似的错误行为 。 如果需要创建一个对象而该对象的主键早已存在于数据库中，IntegrityError异常将会被触发。

这个方法假设进行的是原子操作，并且正确地配置了数据库和正确的底层数据库行为。如果数据库级别没有对`get_or_create`中用到的kwargs强制要求唯一性（unique和unique_together），方法容易导致竞态条件，可能会有相同参数的多行同时插入。（简单理解，kwargs必须指定的是主键或者unique属性的字段才安全。）

最后建议只在Django视图的POST请求中使用get_or_create()，因为这是一个具有修改性质的动作，不应该使用在GET请求中，那样不安全。

可以通过ManyToManyField属性和反向关联使用`get_or_create()`。在这种情况下，应该限制查询在关联的上下文内部。 否则，可能导致完整性问题。

例如下面的模型：

```
class Chapter(models.Model):
    title = models.CharField(max_length=255, unique=True)

class Book(models.Model):
    title = models.CharField(max_length=256)
    chapters = models.ManyToManyField(Chapter)
```

可以通过Book的chapters字段使用`get_or_create()`，但是它只会获取该Book内部的上下文：

```
>>> book = Book.objects.create(title="Ulysses")
>>> book.chapters.get_or_create(title="Telemachus")
(<Chapter: Telemachus>, True)
>>> book.chapters.get_or_create(title="Telemachus")
(<Chapter: Telemachus>, False)
>>> Chapter.objects.create(title="Chapter 1")
<Chapter: Chapter 1>
>>> book.chapters.get_or_create(title="Chapter 1")
# Raises IntegrityError
```

发生这个错误是因为尝试通过Book “Ulysses”获取或者创建“Chapter 1”，但是它不能，因为它与这个book不关联，但因为title 字段是唯一的它仍然不能创建。

在Django1.11在defaults中增加了对可调用值的支持。

### 4. update_or_create()

update_or_create(defaults=None, **kwargs)

类似前面的`get_or_create()`。

**通过给出的kwargs来更新对象的便捷方法， 如果没找到对象，则创建一个新的对象**。defaults是一个由 (field, value)对组成的字典，用于更新对象。defaults中的值可以是可调用对象（也就是说函数等）。

该方法返回一个由(object, created)组成的元组,元组中的object是一个创建的或者是被更新的对象， created是一个标示是否创建了新的对象的布尔值。

`update_or_create`方法尝试通过给出的kwargs 去从数据库中获取匹配的对象。 如果找到匹配的对象，它将会依据defaults 字典给出的值更新字段。

像下面的代码：

```
defaults = {'first_name': 'Bob'}
try:
    obj = Person.objects.get(first_name='John', last_name='Lennon')
    for key, value in defaults.items():
        setattr(obj, key, value)
    obj.save()
except Person.DoesNotExist:
    new_values = {'first_name': 'John', 'last_name': 'Lennon'}
    new_values.update(defaults)
    obj = Person(**new_values)
    obj.save()
```

如果模型的字段数量较大的话，这种模式就变的非常不易用了。 上面的示例可以用`update_or_create()` 重写:

```
obj, created = Person.objects.update_or_create(
    first_name='John', last_name='Lennon',
    defaults={'first_name': 'Bob'},
)
```

kwargs中的名称如何解析的详细描述可以参见`get_or_create()`。

和`get_or_create()`一样，这个方法也容易导致竞态条件，如果数据库层级没有前置唯一性会让多行同时插入。

在Django1.11在defaults中增加了对可调用值的支持。

### 5. bulk_create()

bulk_create(objs, batch_size=None)

以高效的方式（通常只有1个查询，无论有多少对象）将提供的对象列表插入到数据库中：

```
>>> Entry.objects.bulk_create([
...     Entry(headline='This is a test'),
...     Entry(headline='This is only a test'),
... ])
```

注意事项：

- 不会调用模型的save()方法，并且不会发送`pre_save`和`post_save`信号。
- 不适用于多表继承场景中的子模型。
- 如果模型的主键是AutoField，则不会像save()那样检索并设置主键属性，除非数据库后端支持。
- 不适用于多对多关系。

`batch_size`参数控制在单个查询中创建的对象数。

### 6. count()

count()

返回在数据库中对应的QuerySet对象的个数。count()永远不会引发异常。

例如：

```
# 返回总个数.
Entry.objects.count()
# 返回包含有'Lennon'的对象的总数
Entry.objects.filter(headline__contains='Lennon').count()
```

### 7. in_bulk()

in_bulk(id_list=None)

获取主键值的列表，并返回将每个主键值映射到具有给定ID的对象的实例的字典。 如果未提供列表，则会返回查询集中的所有对象。

例如：

```
>>> Blog.objects.in_bulk([1])
{1: <Blog: Beatles Blog>}
>>> Blog.objects.in_bulk([1, 2])
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>}
>>> Blog.objects.in_bulk([])
{}
>>> Blog.objects.in_bulk()
{1: <Blog: Beatles Blog>, 2: <Blog: Cheddar Talk>, 3: <Blog: Django Weblog>}
```

如果向`in_bulk()`传递一个空列表，会得到一个空的字典。

在旧版本中，`id_list`是必需的参数，现在是一个可选参数。

### 8. iterator()

iterator()

提交数据库操作，获取QuerySet，并返回一个迭代器。

QuerySet通常会在内部缓存其结果，以便在重复计算时不会导致额外的查询。而iterator()将直接读取结果，不在QuerySet级别执行任何缓存。对于返回大量只需要访问一次的对象的QuerySet，这可以带来更好的性能，显著减少内存使用。

请注意，在已经提交了的iterator()上使用QuerySet会强制它再次提交数据库操作，进行重复查询。此外，使用iterator()会导致先前的`prefetch_related()`调用被忽略，因为这两个一起优化没有意义。

### 9. latest()

latest(field_name=None)

使用日期字段field_name，按日期返回最新对象。

下例根据Entry的'pub_date'字段返回最新发布的entry：

```
Entry.objects.latest('pub_date')
```

如果模型的Meta指定了`get_latest_by`，则可以将latest()参数留给earliest()或者`field_name`。 默认情况下，Django将使用`get_latest_by`中指定的字段。

earliest()和latest()可能会返回空日期的实例,可能需要过滤掉空值：

```
Entry.objects.filter(pub_date__isnull=False).latest('pub_date')
```

### 10. earliest()

earliest(field_name=None)

类同latest()。

### 11. first()

first()

返回结果集的第一个对象, 当没有找到时返回None。如果QuerySet没有设置排序,则将会自动按主键进行排序。例如：

```
p = Article.objects.order_by('title', 'pub_date').first()
```

first()是一个简便方法，下面的例子和上面的代码效果是一样：

```
try:
    p = Article.objects.order_by('title', 'pub_date')[0]
except IndexError:
    p = None
```

### 12. last()

last()

工作方式类似first()，只是返回的是查询集中最后一个对象。

### 13. aggregate()

aggregate(*args, **kwargs)

返回汇总值的字典（平均值，总和等）,通过QuerySet进行计算。每个参数指定返回的字典中将要包含的值。

使用关键字参数指定的聚合将使用关键字参数的名称作为Annotation 的名称。 匿名参数的名称将基于聚合函数的名称和模型字段生成。 复杂的聚合不可以使用匿名参数，必须指定一个关键字参数作为别名。

例如，想知道Blog Entry 的数目：

```
>>> from django.db.models import Count
>>> q = Blog.objects.aggregate(Count('entry'))
{'entry__count': 16}
```

通过使用关键字参数来指定聚合函数，可以控制返回的聚合的值的名称：

```
>>> q = Blog.objects.aggregate(number_of_entries=Count('entry'))
{'number_of_entries': 16}
```

### 14. exists()

exists()

如果QuerySet包含任何结果，则返回True，否则返回False。

查找具有唯一性字段（例如primary_key）的模型是否在一个QuerySet中的最高效的方法是：

```
entry = Entry.objects.get(pk=123)
if some_queryset.filter(pk=entry.pk).exists():
    print("Entry contained in queryset")
```

它将比下面的方法快很多，这个方法要求对QuerySet求值并迭代整个QuerySet：

```
if entry in some_queryset:
   print("Entry contained in QuerySet")
```

若要查找一个QuerySet是否包含任何元素：

```
if some_queryset.exists():
    print("There is at least one object in some_queryset")
```

将快于：

```
if some_queryset:
    print("There is at least one object in some_queryset")
```

### 15. update()

update(**kwargs)

**对指定的字段执行批量更新操作，并返回匹配的行数**（如果某些行已具有新值，则可能不等于已更新的行数）。

例如，要对2010年发布的所有博客条目启用评论，可以执行以下操作：

```
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False)
```

可以同时更新多个字段 （没有多少字段的限制）。 例如同时更新comments_on和headline字段：

```
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False, headline='This is old')
```

update()方法无需save操作。唯一限制是它只能更新模型主表中的列，而不是关联的模型，例如不能这样做：

```
>>> Entry.objects.update(blog__name='foo') # Won't work!
```

仍然可以根据相关字段进行过滤：

```
>>> Entry.objects.filter(blog__id=1).update(comments_on=True)
```

update()方法返回受影响的行数：

```
>>> Entry.objects.filter(id=64).update(comments_on=True)
1
>>> Entry.objects.filter(slug='nonexistent-slug').update(comments_on=True)
0
>>> Entry.objects.filter(pub_date__year=2010).update(comments_on=False)
132
```

如果你只是更新一下对象，不需要为对象做别的事情，最有效的方法是调用update()，而不是将模型对象加载到内存中。 例如，不要这样做：

```
e = Entry.objects.get(id=10)
e.comments_on = False
e.save()
```

建议如下操作：

```
Entry.objects.filter(id=10).update(comments_on=False)
```

用update()还可以防止在加载对象和调用save()之间的短时间内数据库中某些内容可能发生更改的竞争条件。

如果想更新一个具有自定义save()方法的模型的记录，请循环遍历它们并调用save()，如下所示：

```
for e in Entry.objects.filter(pub_date__year=2010):
    e.comments_on = False
    e.save()
```

### 16. delete()

delete()

批量删除QuerySet中的所有对象，并返回删除的对象个数和每个对象类型的删除次数的字典。

delete()动作是立即执行的。

不能在QuerySet上调用delete()。

例如，要删除特定博客中的所有条目：

```
>>> b = Blog.objects.get(pk=1)
# Delete all the entries belonging to this Blog.
>>> Entry.objects.filter(blog=b).delete()
(4, {'weblog.Entry': 2, 'weblog.Entry_authors': 2})
```

默认情况下，Django的ForeignKey使用SQL约束ON DELETE CASCADE，任何具有指向要删除的对象的外键的对象将与它们一起被删除。 像这样：

```
>>> blogs = Blog.objects.all()
# This will delete all Blogs and all of their Entry objects.
>>> blogs.delete()
(5, {'weblog.Blog': 1, 'weblog.Entry': 2, 'weblog.Entry_authors': 2})
```

这种级联的行为可以通过的ForeignKey的on_delete参数自定义。（什么时候要改变这种行为呢？比如日志数据，就不能和它关联的主体一并被删除！）

delete()会为所有已删除的对象（包括级联删除）发出`pre_delete`和`post_delete`信号。

### 17. as_manager()

classmethod as_manager()

一个类方法，返回Manager的实例与QuerySet的方法的副本。

## 十、Django模型层之字段查询参数

字段查询是指如何指定SQL WHERE子句的内容。它们用作QuerySet的filter(), exclude()和get()方法的关键字参数。

**默认查找类型为exact。**

------

下表列出了所有的字段查询参数：

| 字段名          | 说明                     |
| --------------- | ------------------------ |
| **exact**       | 精确匹配                 |
| **iexact**      | 不区分大小写的精确匹配   |
| **contains**    | 包含匹配                 |
| **icontains**   | 不区分大小写的包含匹配   |
| **in**          | 在..之内的匹配           |
| **gt**          | 大于                     |
| **gte**         | 大于等于                 |
| **lt**          | 小于                     |
| **lte**         | 小于等于                 |
| **startswith**  | 从开头匹配               |
| **istartswith** | 不区分大小写从开头匹配   |
| **endswith**    | 从结尾处匹配             |
| **iendswith**   | 不区分大小写从结尾处匹配 |
| **range**       | 范围匹配                 |
| **date**        | 日期匹配                 |
| **year**        | 年份                     |
| **month**       | 月份                     |
| **day**         | 日期                     |
| **week**        | 第几周                   |
| **week_day**    | 周几                     |
| **time**        | 时间                     |
| **hour**        | 小时                     |
| **minute**      | 分钟                     |
| **second**      | 秒                       |
| **isnull**      | 判断是否为空             |
| search          | 1.10中被废弃             |
| **regex**       | 区分大小写的正则匹配     |
| **iregex**      | 不区分大小写的正则匹配   |

### 1. exact

精确匹配。 默认的查找类型！

```
Entry.objects.get(id__exact=14)
Entry.objects.get(id__exact=None)
```

### 2. iexact

不区分大小写的精确匹配。

```
Blog.objects.get(name__iexact='beatles blog')
Blog.objects.get(name__iexact=None)
```

第一个查询将匹配 'Beatles Blog', 'beatles blog', 'BeAtLes BLoG'等等。

### 3. contains

大小写敏感的包含关系匹配。

```
Entry.objects.get(headline__contains='Lennon')
```

这将匹配标题'Lennon honored today'，但不匹配'lennon honored today'。

### 4. icontains

不区分大小写的包含关系匹配。

```
Entry.objects.get(headline__icontains='Lennon')
```

### 5. in

在给定的列表里查找。

```
Entry.objects.filter(id__in=[1, 3, 4])
```

还可以使用动态查询集，而不是提供文字值列表：

```
inner_qs = Blog.objects.filter(name__contains='Cheddar')
entries = Entry.objects.filter(blog__in=inner_qs)
```

或者从values()或`values_list()`中获取的QuerySet作为比对的对象：

```
inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
entries = Entry.objects.filter(blog__name__in=inner_qs)
```

下面的例子将产生一个异常，因为试图提取两个字段的值，但是查询语句只需要一个字段的值：

```
# 错误的实例，将弹出异常。
inner_qs = Blog.objects.filter(name__contains='Ch').values('name', 'id')
entries = Entry.objects.filter(blog__name__in=inner_qs)
```

### 6. gt

大于

```
Entry.objects.filter(id__gt=4)
```

### 7. gte

大于或等于

### 8. lt

小于

### 9. lte

小于或等于

### 10. startswith

区分大小写，从开始位置匹配。

```
Entry.objects.filter(headline__startswith='Lennon')
```

### 11. istartswith

不区分大小写，从开始位置匹配。

```
Entry.objects.filter(headline__istartswith='Lennon')
```

### 12. endswith

区分大小写，从结束未知开始匹配。

```
Entry.objects.filter(headline__endswith='Lennon')
```

### 13. iendswith

不区分大小写，从结束未知开始匹配。

```
Entry.objects.filter(headline__iendswith='Lennon')
```

### 14. range

范围测试（包含于之中）。

```
import datetime
start_date = datetime.date(2005, 1, 1)
end_date = datetime.date(2005, 3, 31)
Entry.objects.filter(pub_date__range=(start_date, end_date))
```

警告:过滤具有日期的DateTimeField不会包含最后一天，因为边界被解释为“给定日期的0am”。

### 15. date

进行日期对比。

```
Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))
```

当`USE_TZ`为True时，字段将转换为当前时区，然后进行过滤。

### 16. year

对年份进行匹配。

```
Entry.objects.filter(pub_date__year=2005)
Entry.objects.filter(pub_date__year__gte=2005)
```

当`USE_TZ`为True时，在过滤之前，datetime字段将转换为当前时区。

### 17. month

对月份进行匹配。取整数1（1月）至12（12月）。

```
Entry.objects.filter(pub_date__month=12)
Entry.objects.filter(pub_date__month__gte=6)
```

当USE_TZ为True时，在过滤之前，datetime字段将转换为当前时区。

### 18. day

对具体到某一天的匹配。

```
Entry.objects.filter(pub_date__day=3)
Entry.objects.filter(pub_date__day__gte=3)
```

当USE_TZ为True时，在过滤之前，datetime字段将转换为当前时区。

### 19. week

Django1.11中的新功能。根据ISO-8601返回周号（1-52或53），即星期一开始的星期，星期四或之前的第一周。

```
Entry.objects.filter(pub_date__week=52)
Entry.objects.filter(pub_date__week__gte=32, pub_date__week__lte=38)
```

当USE_TZ为True时，字段将转换为当前时区，然后进行过滤。

### 20. week_day

进行“星期几”匹配。 取整数值，星期日为1，星期一为2，星期六为7。

```
Entry.objects.filter(pub_date__week_day=2)
Entry.objects.filter(pub_date__week_day__gte=2)
```

当USE_TZ为True时，在过滤之前，datetime字段将转换为当前时区。

### 21. time

Django1.11中的新功能。

将字段的值转为datetime.time格式并进行对比。

```
Entry.objects.filter(pub_date__time=datetime.time(14, 30))
Entry.objects.filter(pub_date__time__between=(datetime.time(8), datetime.time(17)))
```

USE_TZ为True时，字段将转换为当前时区，然后进行过滤。

### 22. hour

对小时进行匹配。 取0和23之间的整数。

```
Event.objects.filter(timestamp__hour=23)
Event.objects.filter(time__hour=5)
Event.objects.filter(timestamp__hour__gte=12)
```

当USE_TZ为True时，值将过滤前转换为当前时区。

### 23. minute

对分钟匹配。取0和59之间的整数。

```
Event.objects.filter(timestamp__minute=29)
Event.objects.filter(time__minute=46)
Event.objects.filter(timestamp__minute__gte=29)
```

当USE_TZ为True时，值将被过滤前转换为当前时区。

### 24. second

对秒数进行匹配。取0和59之间的整数。

```
Event.objects.filter(timestamp__second=31)
Event.objects.filter(time__second=2)
Event.objects.filter(timestamp__second__gte=31)
```

当USE_TZ为True时，值将过滤前转换为当前时区。

### 25. isnull

值为False或True, 相当于SQL语句IS NULL和IS NOT NULL.

```
Entry.objects.filter(pub_date__isnull=True)
```

### 26. search

自1.10版以来已弃用。

### 27. regex

区分大小写的正则表达式匹配。

```
Entry.objects.get(title__regex=r'^(An?|The) +')
```

建议使用原始字符串（例如，r'foo'而不是'foo'）来传递正则表达式语法。

### 28. iregex

不区分大小写的正则表达式匹配。

```
Entry.objects.get(title__iregex=r'^(an?|the) +')
```

## 十一、聚合函数

Django的`django.db.models`模块提供以下聚合函数。

### 1. expression

引用模型字段的一个字符串，或者一个query expression。

### 2. output_field

用来表示返回值的model field，一个可选的参数。

### 3. `**extra`

关键字参数可以给聚合函数生成的SQL提供额外的信息。

### 4. Avg

class Avg(expression, output_field=FloatField(), **extra)[source]

返回给定表达式的平均值，它必须是数值，除非指定不同的`output_field`。

```
默认的别名：<field>__avg
返回类型：float（或指定任何output_field的类型）
```

### 5. Count

class Count(expression, distinct=False, **extra)[source]

返回与expression相关的对象的个数。

```
默认的别名：<field>__count
返回类型：int
有一个可选的参数：distinct。如果distinct=True，Count 将只计算唯一的实例。默认值为False。
```

### 6. Max

class Max(expression, output_field=None, **extra)[source]

返回expression的最大值。

```
默认的别名：<field>__max
返回类型：与输入字段的类型相同，如果提供则为`output_field`类型
```

### 7. Min

class Min(expression, output_field=None, **extra)[source]

返回expression的最小值。

```
默认的别名：<field>__min
返回类型：与输入字段的类型相同，如果提供则为`output_field`类型
```

### 8. StdDev

class StdDev(expression, sample=False, **extra)[source]

返回expression的标准差。

```
默认的别名：<field>__stddev
返回类型：float
有一个可选的参数：sample。默认情况下，返回群体的标准差。如果sample=True，返回样本的标准差。
SQLite 没有直接提供StdDev。
```

### 9. Sum

class Sum(expression, output_field=None, **extra)[source]

计算expression的所有值的和。

```
默认的别名：<field>__sum
返回类型：与输入字段的类型相同，如果提供则为output_field类型
```

### 10. Variance

class Variance(expression, sample=False, **extra)[source]

返回expression的方差。

```
默认的别名：<field>__variance
返回类型：float
有一个可选的参数：sample。
SQLite 没有直接提供Variance。
```

## 十二、Django测试

本节将简要介绍Django的自动化测试相关内容。

### 一、自动化测试概述

**什么是自动化测试**

测试是一种例行的、不可缺失的工作，用于检查你的程序是否符合预期。

测试可以划分为不同的级别。一些测试可能专注于小细节（比如某一个模型的方法是否会返回预期的值？）， 一些测试则专注于检查软件的整体运行是否正常（用户在对网站进行了一系列的输入后，是否返回了期望的结果？）。

测试可以分为手动测试和自动测试。手动测试很常见，有时候print一个变量内容，都可以看做是测试的一部分。手动测试往往很零碎、不成体系、不够完整、耗时费力、效率低下，测试结果也不一定准确。

自动化测试则是系统地较为完整地对程序进行测试，效率高，准确性高，并且大部分共同的测试工作会由系统来帮你完成。一旦你创建了一组自动化测试程序，当你修改了你的应用，你就可以用这组测试程序来检查你的代码是否仍然同预期的那样运行，而无需执行耗时的手动测试。

**为什么需要测试？**

大家都明白：

- **测试可以节省你的时间**
- **测试不仅仅可以发现问题，还能防止问题**
- **测试使你的代码更受欢迎**
- **测试有助于团队合作**

### 二、编写测试程序

Django是一个全面、完善、严谨的Web框架，当然不会缺少测试功能。

#### 1.**遇见BUG**

很巧，在我们的投票应用中有一个小bug需要修改：在`Question.was_published_recently()`方法的返回值中，当Qeustion在最近的一天发布的时候返回True（这是正确的），然而当Question在未来的日期内发布的时候也返回True（这是错误的）。

我们可以在admin后台创建一个发布日期在未来的Question，然后在shell中验证这个bug：

```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> # 创建一个发布日期在30天后的问卷
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> # 测试一下返回值
>>> future_question.was_published_recently()
True
```

问题的核心在于我们允许创建在未来时间才发布的问卷，由于“未来”不等于“最近”，因此这显然是个bug。

#### 2.**创建一个测试来暴露这个bug**

刚才我们是在shell中测试了这个bug，那如何通过自动化测试来发现这个bug呢？

通常，我们会把测试代码放在应用的`tests.py`文件中，测试系统将自动地从任何名字以test开头的文件中查找测试程序。每个app在创建的时候，都会自动创建一个`tests.py`文件，就像`views.py`等文件一样。

将下面的代码输入投票应用的`polls/tests.py`文件中：

```
import datetime
from django.utils import timezone
from django.test import TestCase
from .models import Question

class QuestionMethodTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        在将来发布的问卷应该返回False
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

我们在这里创建了一个`django.test.TestCase`的子类，它具有一个方法，该方法创建一个`pub_date`在未来的Question实例。最后我们检查`was_published_recently()`的输出，它应该是 False。

#### 3.**运行测试程序**

在终端中，运行下面的命令，

```
$ python manage.py test polls
```

你将看到结果如下：

```
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionMethodTests)
----------------------------------------------------------------------
Traceback (most recent call last):
File "/path/to/mysite/polls/tests.py", line 16, in test_was_published_recently_with_future_question
self.assertIs(future_question.was_published_recently(), False)
AssertionError: True is not False
----------------------------------------------------------------------
Ran 1 test in 0.001s
FAILED (failures=1)
Destroying test database for alias 'default'...
```

这其中都发生了些什么？：

- `python manage.py test polls`命令会查找投票应用中所有的测试程序
- 发现一个`django.test.TestCase`的子类
- 为测试创建一个专用的数据库
- 查找名字以`test`开头的测试方法
- 在`test_was_published_recently_with_future_question`方法中，创建一个Question实例，该实例的pub_data字段的值是30天后的未来日期。
- 然后利用`assertIs()`方法，它发现`was_published_recently()`返回了True，而不是我们希望的False。

最后，测试程序会通知我们哪个测试失败了，错误出现在哪一行。

整个测试用例基本上和Python内置的unittest非常相似，大家可以参考Python教程中测试相关的章节。

#### 3.**修复bug**

我们已经知道了问题所在，现在可以去修复bug了。修改源代码，具体如下：

```
# polls/models.py

def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

再次运行测试程序：

```
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.001s
OK
Destroying test database for alias 'default'...
```

可以看到bug已经没有了。

#### 4.**更加全面的测试**

事实上，前面的测试用例还不够完整，为了使`was_published_recently()`方法更加可靠，我们在上面的测试类中再额外添加两个其它的方法，来更加全面地进行测试。

```
# polls/tests.py

def test_was_published_recently_with_old_question(self):
    """
    只要是超过1天的问卷，返回False
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    最近一天内的问卷，返回True
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

现在我们有三个测试来保证无论发布时间是在过去、现在还是未来`Question.was_published_recently()`都将返回正确的结果。

关于测试的内容，虽然很重要，但是对于刚入门者而言却不是最首要的知识点，等将主体内容掌握后，我们再回头来梳理这一部分。



## 富文本如何使用

from tinymce.models import HTMLField

在settings.py中

```python
TINYMCE_DEFAULT_CONFIG = {
    'theme': 'advanced',
    'width': 600,
    'height': 400,
}
```

在models.py定义的类中

```python
detail=HTMLField(blank=True,...)
```

## views.py中基于类的通用视图的使用

在urls.py中，

```python
url(r'^xx/',类名.as_view(),name='xxx')
```

## Django缓存设置

Settings.py中

```python
# 设置Django框架的缓存
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # 设置django缓存的数据保存在redis数据库中
        "LOCATION": "redis://127.0.0.1:6379/5",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    }
}
```

## 静态文件路径设置

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
#然后在project目录下创建static目录
```

## pymysql驱动 的使用

在init.py中或者settings.py中

```python
import pymysql
pymysql.install_as_MySQLdb()
```

