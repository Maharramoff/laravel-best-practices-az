![sdadsd](img/animated-logo.gif)

Hədəfimiz Laravel üçün SOLID, Dizayn şablonları və s. kimi bəlli təcrübələri təkrarlamaq deyil, əksinə, məhz real Laravel layihələrində nəzərə alınmayan təcrübələri bir araya toplamaqdır.

## Mündəricat

[Tək öhdəlik prinsipi (Single responsibility principle)](#Tək-öhdəlik-prinsipi-single-responsibility-principle)

[İncə kontrollerlər, dolğun modellər](#İncə-kontrollerlər-dolğun-modellər)

[Validasiya](#validasiya)

[Biznes məntiqi xidməti (Service) siniflərdə](#Biznes-məntiqi-xidməti-service-siniflərdə)

[Kodunu təkrarlama (DRY: Don't repeat yourself)](#Kodunu-təkrarlama)

[Sorğu yaradıcısının (query builder) və verilənlər bazasına birbaşa sorğuların əvəzinə Eloquent istifadə edin. Massivlərlə işləmək üçün kolleksiyalara üstünlük verin](#Sorğular-yaradıcısının-query-builder-və-verilənlər-bazasına-birbaşa-sorğuların-əvəzinə-Eloquent-istifadə-edin-massivlərlə-işləmək-üçün-kolleksiyalara-üstünlük-verin)

[Toplu doldurma istifadə edin (mass assignment)](#Toplu-doldurma-istifadə-edin)

[View fayllarında sorğular yazmayın və xəsis yükləmədən istifadə edin (N + 1 problemi)](#View-fayllarında-sorğular-yazmayın-və-xəsis-yükləmədən-istifadə-edin-n--1-problemi)

[Kodlarınızı şərh edin, amma daha da yaxşısı oxunaqlı metod adlarına üstünlük verin](#Kodlarınızı-şərh-edin-amma-daha-da-yaxşısı-oxunaqlı-metod-adlarına-üstünlük-verin)

[Blade Şablonlarında JS və CSS, PHP Kodunda isə HTML yazmayın](#Blade-şablonlarında-js-və-css-php-kodunda-isə-html-yazmayın)

[Kodda mətn yazmaq əvəzinə config, dil sənədləri və sabitlər istifadə edin](#Kodda-mətn-yazmaq-əvəzinə-config-dil-sənədləri-və-sabitlər-istifadə-edin)

[Laravel toplumunun qəbul etdiyi standart vasitələrdən və təcrübələrdən istifadə edin](#Laravel-toplumunun-qəbul-etdiyi-standart-vasitələrdən-və-təcrübələrdən-istifadə-edin)

[Toplumun adlandırma konvensiyalarına riayət edin](#Toplumun-adlandırma-konvensiyalarına-riayət-edin)

[Mümkün olduqca qısa və oxunaqlı sintaksis istifadə edin](#Mümkün-olduqca-qısa-və-oxunaqlı-sintaksis-istifadə-edin)

["new Class" əvəzinə IoC və ya facade istifadə edin](#New-class-əvəzinə-ioc-və-ya-facade-istifadə-edin)

[`.env` sənədindəki məlumatlarla birbaşa işləməyin](#Env-sənədindəki-məlumatlarla-birbaşa-işləməyin)

[Tarixləri standart formatda qeyd edin. Digər formata çevirmək üçün isə accessor və mutatorlardan istifadə edin](#Tarixləri-standart-formatda-qeyd-edin-digər-formata-çevirmək-üçün-isə-accessor-və-mutatorlardan-istifadə-edin)

[Digər tövsiyə və təcrübələr](#Digər-tövsiyə-və-təcrübələr)

### **Tək öhdəlik prinsipi (Single responsibility principle)**

Hər bir sinif və metod ancaq bir işə cavabdeh olmalıdır.

Pis:

```php
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

Yaxşı:

```php
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```

[🔝 Başa qayıt](#Mündəricat)

### **İncə kontrollerlər, dolğun modellər**

Eloquent ilə işləyərkən modellərə, Query Builder və ya standart SQL sorguları ilə işləyərkən isə Repository siniflərinə  üstünlük verin.

Pis:

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

Yaxşı:

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

[🔝 Başa qayıt](#Mündəricat)

### **Validasiya**

İncə kontroller və SRP prinsiplərinə əsasən, validasiya işlərini Request siniflərinə köçürün.

Pis:

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    ....
}
```

Yaxşı:

```php
public function store(PostRequest $request)
{    
    ....
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```

[🔝 Başa qayıt](#Mündəricat)

### **Biznes məntiqi xidməti (Service) siniflərdə**

Kontroller yalnız birbaşa öz vəzifələrini yerinə yetirməlidir, ona görə də biznes məntiqini başqa siniflərə və service siniflərinə köçürün.

Pis:

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    ....
}
```

Yaxşı:

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    ....
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```

[🔝 Başa qayıt](#Mündəricat)

### **Kodunu təkrarlama**

Bu prinsip, bir dəfə yazdığınız kodu mümkün qədər lazım olan hər yerdə istifadə etməyə çağırır. Əgər siz SRP prinsipinə riayət edirsinizsə, təkrarlardan zatən qaçmış olursunuz, amma Laravel də view və bəzi Eloquent sorğularını təkrar istifadə etməyə imkan verir.

Pis:

```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

Yaxşı:

```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```

[🔝 Başa qayıt](#Mündəricat)

### **Sorğu yaradıcısının (query builder) və verilənlər bazasına birbaşa sorğuların əvəzinə Eloquent istifadə edin. Massivlərlə işləmək üçün kolleksiyalara üstünlük verin**

Eloquent maksimum oxunaqlı kod yazmağa imkan verir, onun funksionalığını dəyişmək isə olduqca sadədir. Eloquentdə həmçinin, bir-sıra digər rahat və güclü alətlər var.

Pis:

```php
SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC
```

Yaxşı:

```php
Article::has('user.profile')->verified()->latest()->get();
```

[🔝 Başa qayıt](#Mündəricat)

### **Toplu doldurma istifadə edin**

Pis:

```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;
// Məqaləni kateqoriya ilə əlaqələndir.
$article->category_id = $category->id;
$article->save();
```

Yaxşı:

```php
$category->article()->create($request->validated());
```

[🔝 Başa qayıt](#Mündəricat)

### **View fayllarında sorğular yazmayın və xəsis yükləmədən istifadə edin (N + 1 problemi)**

Pis (100 istifadəçi üçün verilənlər bazasına 101 sorğu gedəcək):

```php
@foreach (User::all() as $user)
    {{ $user->profile->name }}
@endforeach
```

Yaxşı (100 istifadəçi üçün verilənlər bazasına cəmi 2 sorğu gedəcək):

```php
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```

[🔝 Başa qayıt](#Mündəricat)

### **Kodlarınızı şərh edin, amma daha da yaxşısı oxunaqlı metod adlarına üstünlük verin**

Pis:

```php
if (count((array) $builder->getQuery()->joins) > 0)
```

Nisbətən yaxşı:

```php
// Join olub olmadığını müəyyənləşdirin.
if (count((array) $builder->getQuery()->joins) > 0)
```

Yaxşı:

```php
if ($this->hasJoins())
```

[🔝 Başa qayıt](#Mündəricat)

### **Blade Şablonlarında JS və CSS, PHP Kodunda isə HTML yazmayın**

Pis:

```php
let article = `{{ json_encode($article) }}`;
```

Nisbətən yaxşı:

```php
<input id="article" type="hidden" value='@json($article)'>

Və ya

<button class="js-fav-article" data-article='@json($article)'>{{ $article->name }}<button>
```

JavaScript kodu:

```js
let article = $('#article').val();
```

Məlumatları backend-dən frontend-ə ötürmək üçün xüsusi bir paket istifadə etmək daha yaxşı olar.

[🔝 Başa qayıt](#Mündəricat)

### **Kodda mətn yazmaq əvəzinə config, dil sənədləri və sabitlər istifadə edin**

Kodda birbaşa hər-hansı mətn olmamalıdır.

Pis:

```php
public function isNormal()
{
    return $article->type === 'normal';
}

return back()->with('message', 'Sizin məqalə uğurla əlavə olundu!');
```

Yaxşı:

```php
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```

[🔝 Başa qayıt](#Mündəricat)

### **Laravel toplumunun qəbul etdiyi standart vasitələrdən və təcrübələrdən istifadə edin**

Laraveldə tez-tez rast gəlinən məsələlərin həlli üçün standart alətlər mövcuddur. Üçüncü tərəf paketləri və alətlərdən istifadə etmək əvəzinə Laravelin öz alətlərdən istifadə etməyə üstünlük verin. Əks halda sizdən sonra layihəyə gələn Laravel developer, onun üçün yeni olan alətləri öyrənməyə vaxt itirməli, onlarla işləməli və bu alətlərin verə biləcəyi fəsadlarla mübarizə aparmalı olacaq. Bu səbəbdən toplumdan kömək almaq da çətin olacaq. Müştərinizi və ya işəgötürəninizi velosipedləriniz üçün əvəz ödəməyə məcbur etməyin.

Məsələ | Standart alət | Qeyri-standart alət
------------ | ------------- | -------------
Səlahiyyətlər | Policies | Entrust, Sentinel və digər paketlər və ya şəxsi həll
JS, CSS və s. ilə işlər | Laravel Mix | Grunt, Gulp, kənar paketlər
Proqramlaşdırma mühiti | Homestead | Docker
Tətbiqin yerləşdirilməsi | Laravel Forge | Deployer və digərləri
Testləmə | Phpunit, Mockery | Phpspec
e2e testləmə | Laravel Dusk | Codeception
VB ilə iş | Eloquent | SQL, Query Builder, Doctrine
Şablonlar | Blade | Twig
Məlumatlarla iş | Laravel Collections | Massivlər
Form validasiyaları | Request sinifləri | Kənar paketlər, kontroller daxili validasiya
Daxil olma | Daxili funksional | Kənar paketlər, şəxsi həll
API ilə daxil olma | Laravel Passport | JWT işlədən kənar paketlər, OAuth
API yaratma | Daxili funksional | Dingo API və digər paketlər
VB strukturu ilə iş | Miqrasiyalar | VB ilə birbaşa işləmək
Lokalizasiya | Daxili funksional | Kənar paketlər
Real zamanda məlumatlarla iş | Laravel Echo, Pusher | Kənar paketlər və web-socketlərlə birbaşa işləmək
Test dataların generasiyası | Seeder sinifləri, model fabrikləri, Faker | Manual daxil etmə və kənar paketlər
Tapşırıq planlaması | Laravelin daxili funksionalı | Skriptlər və kənar paketlər
VB | MySQL, PostgreSQL, SQLite, SQL Server | MongoDb

[🔝 Başa qayıt](#Mündəricat)

### **Toplumun adlandırma konvensiyalarına riayət edin**

 Kod yazarkən [PSR standartlarına](http://www.php-fig.org/psr/psr-2/) əməl edin.
 
 Həmçinin adlandırma ilə bağlı digər razılıqlara da əməl edin:

Nə | Qayda | Qəbul olunur | Qəbul olunmur
------------ | ------------- | ------------- | -------------
Kontroller | təklik | ArticleController | ~~ArticlesController~~
Marşrutlar | cəmlik | articles/1 | ~~article/1~~
Marşrut adları | snake_case | users.show_active | ~~users.show-active, show-active-users~~
Model | təklik | User | ~~Users~~
hasOne və belongsTo əlaqələri | təklik | articleComment | ~~articleComments, article_comment~~
Digər əlaqələr | cəmlik | articleComments | ~~articleComment, article_comments~~
Cədvəl | cəmlik | article_comments | ~~article_comment, articleComments~~
Pivot cədvəl | təklik (model adları əlifba sırası ilə) | article_user | ~~user_article, articles_users~~
Cədvəl sütunu | snake_case (model adı olmadan) | meta_title | ~~MetaTitle; article_meta_title~~
Model dəyişənləri | snake_case | $model->created_at | ~~$model->createdAt~~
Kənar açarlar | model adı təklikdə və _id | article_id | ~~ArticleId, id_article, articles_id~~
Əsas açar | - | id | ~~custom_id~~
Miqrasiya | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
Metod | camelCase | getAll | ~~get_all~~
Resurs kontrollerlərində metod | [cədvəl](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
Test metodları | camelCase | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
Dəyişənlər | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
Kolleksiyalar | təsvir edici cəmlikdə | $activeUsers = User::active()->get() | ~~$active, $data~~
Obyekt | təsvir edici təklikdə | $activeUser = User::active()->first() | ~~$users, $obj~~
Config və dil fayllarında indekslər | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
View | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Config fayl | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Kontrakt (interface) | sifət və ya isim | Authenticatable | ~~AuthenticationInterface, IAuthentication~~
Trait | sifət | Notifiable | ~~NotificationTrait~~

[🔝 Başa qayıt](#Mündəricat)

### **Mümkün olduqca qısa və oxunaqlı sintaksis istifadə edin**

Pis:

```php
$request->session()->get('cart');
$request->input('name');
```

Yaxşı:

```php
session('cart');
$request->name;
```

Digər nümunələr:

Tez-tez istifadə olunan sintaksis | Daha yığcam və oxunaqlı sintaksis
------------ | -------------
`Session::get('cart')` | `session('cart')`
`$request->session()->get('cart')` | `session('cart')`
`Session::put('cart', $data)` | `session(['cart' => $data])`
`$request->input('name'), Request::get('name')` | `$request->name, request('name')`
`return Redirect::back()` | `return back()`
`is_null($object->relation) ? null : $object->relation->id` | `optional($object->relation)->id`
`return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))`
`$request->has('value') ? $request->value : 'default';` | `$request->get('value', 'default')`
`Carbon::now(), Carbon::today()` | `now(), today()`
`App::make('Class')` | `app('Class')`
`->where('column', '=', 1)` | `->where('column', 1)`
`->orderBy('created_at', 'desc')` | `->latest()`
`->orderBy('age', 'desc')` | `->latest('age')`
`->orderBy('created_at', 'asc')` | `->oldest()`
`->select('id', 'name')->get()` | `->get(['id', 'name'])`
`->first()->name` | `->value('name')`

[🔝 Başa qayıt](#Mündəricat)

### **"new Class" əvəzinə IoC və ya facade istifadə edin**

Sinflərin "new Class" Sintaksisi ilə tətbiq edilməsi, tətbiq hissələri arasında güclü bir bağ yaradır və testi çətinləşdirir. Bunun üçün **IoC** konteyner və ya **facade** istifadə edin.

Pis:

```php
$user = new User;
$user->create($request->validated());
```

Yaxşı:

```php
public function __construct(User $user)
{
    $this->user = $user;
}

....

$this->user->create($request->validated());
```

[🔝 Başa qayıt](#Mündəricat)

### **`.env` sənədindəki məlumatlarla birbaşa işləməyin**

Məlumatları `.env` faylından konfiqurasiya faylına köçürün və bu məlumatları istifadə etmək üçün tətbiqdə 'config ()' istifadə edin.

Pis:

```php
$apiKey = env('API_KEY');
```

Yaxşı:

```php
// config/api.php
'key' => env('API_KEY'),

// Tətbiqin konfiqurasiya faylını istifadə edin
$apiKey = config('api.key');
```

[🔝 Başa qayıt](#Mündəricat)

### **Tarixləri standart formatda qeyd edin. Digər formata çevirmək üçün isə accessor və mutatorlardan istifadə edin**

Pis:

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

Yaxşı:

```php
// Model
protected $dates = ['ordered_at', 'created_at', 'updated_at'];
// Oxucu (accessor)
public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// Şablon
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```

[🔝 Başa qayıt](#Mündəricat)

### **Digər tövsiyə və təcrübələr**

Marşrutlarda məntiq yazmayın.

Blade şablonlarında çalışın standart PHP istifadə etməyin.

[🔝 Başa qayıt](#Mündəricat)