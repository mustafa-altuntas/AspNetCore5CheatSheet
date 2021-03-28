# NOT:
string.IsNullOrEmpty(myData)

# AspNetCore5CheatSheet

# Kullanıcıdan Veri Alma Yöntemleri
![](img\screenshot_20210120_192859.png)
## Form Üzerinden Veri Alma

```c#     
[HttpPost]
public IActionResult GetProducts(IFormCollection datas)
{
    var value1 = datas["txtValue1"];
    var value2 = datas["txtValue2"];
    var value3 = datas["txtValue3"];
    return View();
}
```
```c#
[HttpPost]
public IActionResult GetProducts(string txtValue1, stxtValue2, string txtValue3)
{
    var value1 = txtValue1;
    var value2 = txtValue2;
    var value3 = txtValue3;
    return View();
}
```

```c#
public class MyData
{
    public string txtValue1 { get; set; }
    public string txtValue2 { get; set; }
    public string txtValue3 { get; set; }
}

***************************************************

[HttpPost]
public IActionResult GetProducts(MyData myData)
{
    var value1 = myData.txtValue1;
    var value2 = myData.txtValue2;
    var value3 = myData.txtValue3;
    return View();
}

```
---

## Query String Üzerinden Veri Alma
Güvenlik gerektirmeyen bilgileri url üzerinde tasinmasi icin kullanilan yapilanmadır.

https://......com/sehir/ankara?queryString

https://......com/sehir/ankara?ilce=2


```
https://localhost:5001/Product/VeriAl?a=merhaba&b=5
```
```c#
public IActionResult VeriAl(string a, string b)
{
    return View();
}
```
```c#
public IActionResult VeriAl()
{
    var queyString = HttpContext.Request.QueryString;
    var a = Request.Query["a"].ToString();
    var b = Request.Query["b"].ToString();
    return View();
}
```


```c#
https://localhost:5001/Product/VeriAl?a=merhaba&b=5


public class MyQueryData
{
    public string A { get; set; }
    public string B { get; set; }
}

 
public IActionResult VeriAl(MyQueryData myData)
{
    var a = myData.A;
    var b = myData.B;
    return View();
}
```
---

## Route Parameter Üzerinden Veri Alma

```
https://localhost:5001/Product/VeriAl/5
```
```c#
public IActionResult VeriAl(string id)
{
    return View();
}
```



``` c#
//Gelen verileri Request propertisi ile ala.

public IActionResult VeriAl(string id)
{
    var values = Request.RouteValues;
    return View();
}
```

---

## Header Üzerinden Veri Alma
Http isteğinde bulunan veri
Haderler genellikle ilgili istekla alakalı nitelikleri barındırır.


``` c#
public IActionResult VeriAl()
{
    var headers = Request.Headers.ToList();
    return View(); 
}

```
---

## Ajax Tabanlı Veri Alma
Client tabanlı istek yapmamızı sağlayan ve bu isteklerin sonucunu almamızı sağlayan bir js temelli mimaridir.


``` html
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>

<button id="btnGonder" >Gönder</button>

<script>
    $("#btnGonder").click(() => {
        $.post("https://localhost:5001/product/verial", { a:"A data", b:"B data" });
    });
</script>

```

``` c#
public class AjaxData
{
    public string A { get; set; }
    public string B { get; set; }
}

**********************************************

[HttpPost]
public IActionResult VeriAl(AjaxData ajaxData)
{
    return View();
}

public IActionResult CreateProduct()
{
    return View();
}
```
---

</br>
</br>
</br>
</br>


# Tuple Nesne Post Etme ve Yakalama


``` html

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@model (WebApplication1.Models.Product product, WebApplication1.Models.User user)

<form asp-action="CreateProduct" asp-controller="Product" method="post">
    <input type="type" asp-for="product.Name" />   <br />
    <input type="type" asp-for="user.Name" /><br />
    <button>Gönder</button>
</form>


```

``` c#

[HttpPost]
public IActionResult CreateProduct([Bind(Prefix ="Item1")] Product product, [Bind(Prefix ="Item2")] User user)
{

    return View();
}

public IActionResult CreateProduct()
{
    var tuple = (new Product(), new User());
    return View(tuple);
}

```



# Kullanıcıdan Gelen Verilerin Doğrulanamsı Validations
Bir değerin içinde bulunduğu şartlara uygun olması durumudur. Belirlenen koşullara ve amaca uygun olup olmama durumunun kontrol edilmesi. Bu kontrollere göre verinin işleme tabi tutulması durumudur.

Server tarafında yapılan validation işlemine Server Side Validation denir.

Cleint tarafında yapılan validation işlemine Cleint Side Validation denir.

Gelen her veriyi
- alacakmıyız
- işleme tabi tutacakmıyız
- veritabanına kaydedecekmiyiz


Validation ;
- bind olma
- modulbazlı
- yöntemleri
- dataanothesions
- fluentvalidation


``` c#
public class Product
{
    [Required(ErrorMessage ="Lütfen product name'i girinzi.")]
    [StringLength(100,ErrorMessage ="Name'i en fazla 100 karakter giriniz.")]
    public string Name { get; set; }
    public int Quantity { get; set; }
    [EmailAddress(ErrorMessage ="Email formatı hatalı")]
    public string Email { get; set; }
}
```


``` html
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@model WebApplication1.Models.Product

<div asp-validation-summary="All"></div>
<form asp-action="CreateProduct" asp-controller="Product" method="post">
    <input type="type" asp-for="Name" placeholder="Product Name" />
    <span asp-validation-for="Name"></span> <br />

    <input type="type" asp-for="Quantity" placeholder="Quantity" />
    <span asp-validation-for="Quantity"></span> <br />

    <input type="type" asp-for="Email" placeholder="Email" />
    <span asp-validation-for="Email"></span> <br />
    <button>Gönder</button>
</form>
```


``` c#
public class ProductController : Controller
{
    public IActionResult CreateProduct()
    {
        return View();
    }

    [HttpPost]
    public IActionResult CreateProduct(Product product)
    {
        //ModelState : MVC'de bir type'ın data annotationslarının durumunu kontrol eden ve geriye sonuç döen bir propertiy.
        if (!ModelState.IsValid)
        {
            //Loglama
            //Kullanıcı ilgilendirme
            return View(product);
        }
        //İşlem/Operasyon/Algoritmaya tabi tutulur.
        return View();
    }
}

```

## ModelMetadataType ile Validation Sorumluluğunu Başka Bir Sınıfa Yükleme

Model klasörümüzün içerisinde ModelMetadataType adında yeni bir klasör açarak validasyon sınıflarımızı bu klasörde oluşturacağız.
ModelMetadataType klasörümüzün içerisinde Product sınıfımız için ProductMetadata.cs sınıfımızı oluşturuyoruz ve bu sınıfımız içerisinde validasyon uygulanacak propertileri tanımlıyoruz.

``` c#
namespace WebApplication1.Models.ModelMetaDataType
{
    public class ProductMetaData
    {
        [Required(ErrorMessage = "Lütfen product name'i girinzi.")]
        [StringLength(100, ErrorMessage = "Name'i en fazla 100 karakter giriniz.")]
        public int Quantity { get; set; }
        [EmailAddress(ErrorMessage = "Email formatı hatalı")]
        public string Email { get; set; }
    }
}

```

Şimdide bu sınıfımızı Product.cs sınıfımıza uygulayalım.

``` c#
    [ModelMetadataType(typeof(ProductMetaData)]
    public class Product
    {
        public string Name { get; set; }
        public int Quantity { get; set; }
        public string Email { get; set; }
    }
```

Artık Product nesnemiz sade ve anlaşılır ve validation işlemleri tanımlı bir nesnedir.

## FluentValidation Kütüphanesi ile Validation İşlemleri

`FluentValidation.AspNetCore` kütüphanesini indiriyoruz.
Kütüphaneyi indirdikten sonra Startup.cs dosyasında bu değişikliği yapıyoruz.

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews().AddFluentValidation(x =>x.RegisterValidatorsFromAssemblyContaining<Startup>());
}
```

Projemize FluentValidator servisini ekledikten sonra Model klasörümüzde Validatörlerimizi tutacağımız bir `Validators` klasörü oluşturuyoruz.
Validators klasörümğüzde Product nesnemiz için `ProductValidator.cs` class ımızı oluşturuyoruz.

``` c#
namespace WebApplication1.Models.Validators
{
    public class ProductValidator : AbstractValidator<Product>
    {
        public ProductValidator()
        {
            RuleFor(x => x.Email).NotEmpty().NotNull().WithMessage("Eposta alanı boş bırakılamaz.");
            RuleFor(x => x.Email).EmailAddress().WithMessage("Eposta formatı hatalı.");
            RuleFor(x => x.Name).NotNull().NotEmpty().WithMessage("Name alanı boş bıakılamaz.");
            RuleFor(x => x.Name).MinimumLength(5).WithMessage("Name alanı en az 5 karakter olmalıdır.");
        }
    }
}

```

## Server'da ki Validation'ları Dinamik Olarak Client Tabanlı Uygulamak

Öncelikle projemizde `wwwroot` adında bir klasör oluşturuyoruz ve klasörün üzerine sağ tıklayarak açaılan seçeneklerden `add`'in altında `Client-Side-Library` seçiyoruz.
`Client-Side-Library` ile şu üç kütüphaneyi ;
- jquery
- jquery-validate
- jquery-validation-unobtrusive
projemize dağil ediyoruz. Projemize dağil ettiğimiz bu kütüphaneleri html sayfalarımıza ekliyoruz.
``` html
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@model WebApplication1.Models.Product

<script src="~/jquery/jquery.js"></script>
<script src="~/jquery-validate/jquery.validate.js"></script>
<script src="~/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>

<div asp-validation-summary="All"></div>
<form asp-action="CreateProduct" asp-controller="Product" method="post">
    <input type="type" asp-for="Name" placeholder="Product Name" />
    <span asp-validation-for="Name"></span> <br />

    <input type="type" asp-for="Quantity" placeholder="Quantity" />
    <span asp-validation-for="Quantity"></span> <br />

    <input type="type" asp-for="Email" placeholder="Email" />
    <span asp-validation-for="Email"></span> <br />
    <button>Gönder</button>
</form>

```


#  Filterları Kontrol Etmeyi Kapatma Kapatma
ModelState hatalarını kontrol etmesini kapatıyoruz. Çünkü kendi `ValidationFilter`'ımızı yazacağız.
Startup.cs dosyasında ``ConfigureServices()`` methoduna şunları yazacağız;


``` c#
services.Configure<ApiBehaviorOptions>(options =>
{
    options.SuppressModelStateInvalidFilter = true;
});
```
Hata kontrol mekanizmasını kapattığımıza göre kendi mekanizmamızı oluşturmaya abşlayabiliriz.
## 1) ErrorDto class'ımizi oluşturalım
```c#
    public class ErrorDto
    {
        public List<string> Errors { get; set; }
        public int Status { get; set; }
        //Hatanın durum kodu
    }
```
```c#
    public class ProductDto
    {
        public int Id { get; set; }
        [Required(ErrorMessage ="{0} Zorunludur")]
        public string Name { get; set; }
        [Range(1,int.MaxValue,ErrorMessage ="{0} alanı 1'den büyük bir değer olmalıdır.")]
        public int Stock { get; set; }
        [Range(1, double.MaxValue, ErrorMessage = "{0} alanı 1'den büyük bir değer olmalıdır.")]
        public decimal Price { get; set; }
    }
```
## 2) Fillet oluşturma
Önce `Filters` adında bir `klasör` oluşturuyoruz ve ardında bu klasörün içerisinde ValidationFilter adında bir `class` oluşturuyoruz. Oluşturduğumuz filtir'ın `ActionFilter` olması için oluşturduğumuz class'ı `ActionFilterAttribute` sınıfından türetiyoruz.
Dahasonra bu `Action` methoda request gelirken nerede müdehale edeceğimi `override` methodlardan seçebilirim. </br>
Bu seçenekler ;
| metod   | Açıklama |
| ----------- | ----------- |
| OnActionExecuted      | Actionç alışması bitttiği zaman       |
| OnActionExecuting     | Action çalışıyorken                   |
| OnActionExecutionAsync| Action çalışması async olduğu zaman   |
| OnResultExecuted      | sonuç çalıştığı zaman                 |
| OnResultExecuting     | sonuç çalışıyorken                    |

Biz burada ilk anda müdahale edeceğiz yani bir request ilgili `action`'a geldiği zaman.
```c#
public class ValidationFilter : ActionFilterAttribute
    {
        public override void OnActionExecuting(ActionExecutingContext context)
        {
            if (!context.ModelState.IsValid)
            {
                ErrorDto errorDto = new ErrorDto();
                errorDto.Status = 400;

                IEnumerable<ModelError> modelErrors = context.ModelState.Values.SelectMany(v => v.Errors);

                modelErrors.ToList().ForEach(x =>
                {
                    errorDto.Errors.Add(x.ErrorMessage.ToString());
                });

                context.Result = new BadRequestObjectResult(errorDto);
            }

        }
    }
```

## 3) Filter'ı controllerda kullanma
Ben bu filter'ı `ProductsController` da `Save` methodumda kullanacağım 

```c#
[ValidationFilter]
[HttpPost]
public async Task<IActionResult> Save(ProductDto vm)
{
    var model = await _productService.AddAsync(_mapper.Map<Product>(vm));
    return Created(string.Empty, _mapper.Map<ProductDto>(model));
}
```

# NotFoudFilter eklema

# 1) NotFountFilter class'ını oluşturma

```c#
public class NotFoundFilter: ActionFilterAttribute
    {
        private readonly IProductService _productService;

        public NotFoundFilter(IProductService productService)
        {
            _productService = productService;
        }

        public async override Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            int id = (int) context.ActionArguments.Values.FirstOrDefault();
            var product = await _productService.GetByIdAsync(id);

            if (product!=null)
            {
                await next();
            }
            else
            {
                ErrorDto errorDto = new ErrorDto();
                errorDto.Status = 404;
                errorDto.Errors.Add($"id'si {id} olana ürün veritabanında bulunamadı");

                context.Result = new NotFoundObjectResult(errorDto); 
            }
        }
    }
```
## Not :
Oluşturduğumuz ``NotFoundFilter`` içerisinde bir ``DI`` nesnesi aldığından dolayı biz bunu önce ``Startup.cs`` de ``ConfigureServices()``  methodunda tanımlamalıyız ardından da bu filter'ı kullanacağımız yerde `ServiceFilter()`  olarak tanımlamalıyız.

## 2) Oluşturduğumuz filter DI nesnesi içerdiği için Startup.cs içinde tanımlama

```c#
services.AddScoped<NotFoundFilter>();
```

## 3) NotFoundFiller'ı ProductsController'da kullanama

```c#
[ServiceFilter(typeof(NotFoundFilter))]
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id)
{
    var model = await _productService.GetByIdAsync(id);
    return Ok(_mapper.Map<ProductDto>(model));
}
```


# Hataları Global düzeyde ele alma - 1
Bu işlemi `Startup.cs` de midlaware 'lar ile çözeceğiz.


```c#
app.UseExceptionHandler(config =>
{
    config.Run(async context =>
    {
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        var error = context.Features.Get<IExceptionHandlerFeature>();
        if (error != null)
        {
            var ex = error.Error;
            ErrorDto errorDto = new ErrorDto();
            errorDto.Status = 500;
            errorDto.Errors.Add(ex.Message);
            await context.Response.WriteAsync(JsonConvert.SerializeObject(errorDto));
        }
    });
});
```


# Hataları Global düzeyde ele alma - 2
Bu işlemi ``Extension`` method ile yapacağiz ve ``Startup.cs`` içerisinde kullanacağiz.
Şimdi `Extensions` adında bir klasör oluşturuyorum ve bu klasöründe içinde `UseCostomeExceptionHandle` adıda bir class oluşturuyorum bu class'ın ``static`` olması gerekiyor.

```c#
public static class UseCostomeExceptionHandle
    {
        public static void UseCostomeException(this IApplicationBuilder app)
        {

            app.UseExceptionHandler(config =>
            {
                config.Run(async context =>
                {
                    context.Response.StatusCode = 500;
                    context.Response.ContentType = "application/json";
                    var error = context.Features.Get<IExceptionHandlerFeature>();
                    if (error != null)
                    {
                        var ex = error.Error;
                        ErrorDto errorDto = new ErrorDto();
                        errorDto.Status = 500;
                        errorDto.Errors.Add(ex.Message);
                        await context.Response.WriteAsync(JsonConvert.SerializeObject(errorDto));
                    }
                });
            });

        }
    }
```

Bu işlemlerin ardından şimdi ``Startup.cs`` dosyamın içinde ``Configure()`` methodunda bu ``Extension``'umu çağıraağım.

```c#
UseCostomeExceptionHandle.UseCostomeException(app);
```


# DropDownList - Ajax JsonResult Kullaımı
Bu başlık aldında senaryomuz şuşekilde olacak veritabanında şehirler ve bu şehirlere ait ilçeler olacak ve bide sayfa açıldığında alt alta iki tane selectlist oluşturacağız birinde iller diğerinde ilçeleri lisleliyeceğiz. İlçeler il seçmeden listelenmiyecek ve il seçtiğimizde ilçeleri ajax ile sayfamıza getireceğiz.

İlk önce modellerimiz oluşturalım ``Sehir.cs``, ``Ilce.cs`` ve ``HomeViewModel.cs`` adlarında 3 class oluşturuyoruz. Sehir.cs ve Ilce.cs veri tabanımızdaki tablolarımıza karşılık geliyor tamam onu anladık da HomeViewModel.cs ne için dediğinizi duar gibiyim hemen cevaplayayım HomeViewModel.cs class'ımızda indek sayfasında selectList'imizde seçim yaptığımızda dönecek SehirId ve IlceId 'yi tutacağız. Modellerim suşekilde ...

```c#
public class Ilce
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public int SehirId { get; set; }
        public virtual Sehir Sehir { get; set; }

    }
```

```c#
    public class Sehir
    {
        public Sehir()
        {
            Ilces = new List<Ilce>();
        }
        public int Id { get; set; }
        public string Name { get; set; }

        public virtual List<Ilce> Ilces { get; set; }

    }
```

```c#
    public class HomeViewModel
    {
        public int SehirId { get; set; }
        public int IlceId { get; set; }
    }
```

Evet modellerimizi oluşturduk şimdide HomeController'umuzdaki index methodunu kodlayalım. Burada ilk olarak index sayfamıza şehirleri getiriyoruz. Daha sonra kullanıcı bir şehir seçtiğinde ajax ile Veriler() methodunu çalıştırıyoruz ve veritabanından sayfamıza ilçeleri çekiyoruz.

```c#
        private readonly AppDbContext _context;
        public HomeController(AppDbContext context)
        {
            _context = context;
        }
        public IActionResult Index()
        {
            var sehirler = _context.Sehirs.ToList();
            ViewBag.Sehirler = new SelectList(sehirler, "Id", "Name");
            return View();
        }

        public JsonResult Veriler(int SehirId)
        {
            var veriler = _context.Ilces.Where(x=>x.SehirId==SehirId).ToList();

            return Json(veriler);
        }
```

Şimdide html tarafına bakalım index.cshtml sayfasına bakalım.
```c#
@model HomeViewModel

<br />
<br />
<div class="container">
    <div class="form-group">
        @if (ViewBag.Sehirler != null)
        {
            @Html.DropDownListFor(model => model.SehirId, ViewBag.Sehirler as SelectList, "--Şehir Seç", new { @class = "form-control" })
        }
    </div>

    <div class="form-group">
        @Html.DropDownListFor(model => model.IlceId, new SelectList(""), "--İlçe Seç", new { @class = "form-control" })
    </div>

</div>

//Ajax'ın çalışması için jquery kütüphanesini projemize dahil ediyoruz.
<script src="~/lib/jquery/dist/jquery.js"></script>

<script>
    $("#SehirId").change(function () {
        $.ajax({
            type: "GET",
            url: '@Url.Action("Veriler")',
            data: { SehirId: $("#SehirId").val() },
            dataType: "json",
            success: function (data) {
                $("#IlceId").empty;
                data.forEach(function (item) {
                    $("#IlceId").append("<option value='" +item.id+"' >"+item.name+"</option>")
                })

            }
        })
    })
    
</script>
```


# ModelState 'e hata ekleme -- error ekleme


```c#
    ModelState.AddModelError("", item.Description);
    ModelState.AddModelError("Hedef input", "Hata mesajı");
```

