![sdadsd](img/logo-full.gif)

Hədəfimiz Laravel üçün SOLID, Dizayn şablonları və s. kimi bəlli təcrübələri təkrarlamaq deyil, əksinə, məhz real Laravel layihələrində nəzərə alınmayan təcrübələri bir araya toplamaqdır.

## Mündəricat

[Tək öhdəlik prinsipi (Single responsibility principle)](#Tək-öhdəlik-prinsipi-single-responsibility-principle)

[İncə kontrollerlər, dolğun modellər](#İncə-kontrollerlər-dolğun-modellər)

[Validasiya](#validasiya)

[Biznes məntiqi xidməti siniflərdə](#Biznes-məntiqi-xidməti-siniflərdə)

[Özün-özünü təkrarlama (DRY: Don't repeat yourself)](#Özün-özünü-təkrarlama)

[Sorğular konstruktorundan (query builder) və verilənlər bazasına birbaşa sorğulardan daha çox Eloquentə üstünlük verin. Massivlərlə işləmək üçün kolleksiyalara üstünlük verin](#Sorğular-konstruktorundan-query-builder-və-verilənlər-bazasına-birbaşa-sorğulardan-daha-çox-Eloquentə-üstünlük-verin-massivlərlə-işləmək-üçün-kolleksiyalara-üstünlük-verin)

[Toplu doldurma istifadə edin (mass assignment)](#Toplu-doldurma-istifadə-edin)

[View fayllarında sorğular yazmayın və xəsis yükləmədən istifadə edin (N + 1 problemi)](#View-fayllarında-sorğular-yazmayın-və-xəsis-yükləmədən-istifadə-edin-N-1-problemi)

[Kodlarınızı şərh edin, amma daha da yaxşısı oxunaqlı metod adlarına üstünlük verin](#Kodlarınızı-şərh-edin-amma-daha-da-yaxşısı-oxunaqlı-metod-adlarına-üstünlük-verin)

[Blade Şablonlarında JS və CSS, PHP Kodunda isə HTML yazmayın](#Blade-şablonlarında-js-və-css-php-kodunda-isə-html-yazmayın)

[Kodda mətn yazmaq əvəzinə config, dil sənədləri və sabitlər istifadə edin](#Kodda-mətn-yazmaq-əvəzinə-config-dil-sənədləri-və-sabitlər-istifadə-edin)

[Laravel toplumunun qəbul etdiyi standart vasitələrdən və təcrübələrdən istifadə edin](#Laravel-toplumunun-qəbul-etdiyi-standart-vasitələrdən-və-təcrübələrdən-istifadə-edin)

[Toplumun adlandırma konvensiyalarına riayət edin](#Toplumun-adlandırma-konvensiyalarına-riayət-edin)

[Mümkün olduqca qısa və oxunaqlı sintaksis istifadə edin](#Mümkün-olduqca-qısa-və-oxunaqlı-sintaksis-istifadə-edin)

["new Class" əvəzinə IoC və ya facade istifadə edin](#New-Class-əvəzinə-IoC-və-ya-facade-istifadə-edin)

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

### **Biznes məntiqi xidməti siniflərdə**

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

### **Özün-özünü təkrarlama**

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

### **Sorğular konstruktorundan (query builder) və verilənlər bazasına birbaşa sorğulardan daha çox Eloquentə üstünlük verin. Massivlərlə işləmək üçün kolleksiyalara üstünlük verin**

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
// Привязать статью к категории.
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
// Determine if there are any joins.
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

Laravel имеет встроенные инструменты для решения часто встречаемых задач. Предпочитайте пользоваться ими использованию сторонних пакетов и инструментов. Laravel разработчику, пришедшему в проект после вас, придется изучать и работать с новым для него инструментом, со всеми вытекающими последствиями. Получить помощь от сообщества будет также гораздо труднее. Не заставляйте клиента или работодателя платить за ваши велосипеды.
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

### **Соблюдайте соглашения сообщества об именовании**

 Следуйте [стандартам PSR](http://www.php-fig.org/psr/psr-2/) при написании кода.
 
 Также, соблюдайте другие cоглашения об именовании:

Что | Правило | Принято | Не принято
------------ | ------------- | ------------- | -------------
Контроллер | ед. ч. | ArticleController | ~~ArticlesController~~
Маршруты | мн. ч. | articles/1 | ~~article/1~~
Имена маршрутов | snake_case | users.show_active | ~~users.show-active, show-active-users~~
Модель | ед. ч. | User | ~~Users~~
Отношения hasOne и belongsTo | ед. ч. | articleComment | ~~articleComments, article_comment~~
Все остальные отношения | мн. ч. | articleComments | ~~articleComment, article_comments~~
Таблица | мн. ч. | article_comments | ~~article_comment, articleComments~~
Pivot таблица | имена моделей в алфавитном порядке в ед. ч. | article_user | ~~user_article, articles_users~~
Столбец в таблице | snake_case без имени модели | meta_title | ~~MetaTitle; article_meta_title~~
Свойство модели | snake_case | $model->created_at | ~~$model->createdAt~~
Внешний ключ | имя модели ед. ч. и _id | article_id | ~~ArticleId, id_article, articles_id~~
Первичный ключ | - | id | ~~custom_id~~
Миграция | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
Метод | camelCase | getAll | ~~get_all~~
Метод в контроллере ресурсов | [таблица](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
Метод в тесте | camelCase | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
Переменные | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
Коллекция | описательное, мн. ч. | $activeUsers = User::active()->get() | ~~$active, $data~~
Объект | описательное, ед. ч. | $activeUser = User::active()->first() | ~~$users, $obj~~
Индексы в конфиге и языковых файлах | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
Представление | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Конфигурационный файл | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Контракт (интерфейс) | прилагательное или существительное | Authenticatable | ~~AuthenticationInterface, IAuthentication~~
Трейт | прилагательное | Notifiable | ~~NotificationTrait~~

[🔝 Başa qayıt](#Mündəricat)

### **Короткий и читаемый синтаксис там, где это возможно**

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

Еще примеры:

Часто используемый синтаксис | Более короткий и читаемый синтаксис
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

### **Используйте IoC или фасады вместо new Class**

Внедрение классов через синтаксис new Class создает сильное сопряжение между частями приложения и усложняет тестирование. Используйте контейнер или фасады.

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

### **Не работайте с данными из файла `.env` напрямую**

Передайте данные из `.env` файла в кофигурационный файл и используйте `config()` в приложении, чтобы использовать эти данными.

Pis:

```php
$apiKey = env('API_KEY');
```

Yaxşı:

```php
// config/api.php
'key' => env('API_KEY'),

// Используйте данные в приложении
$apiKey = config('api.key');
```

[🔝 Başa qayıt](#Mündəricat)

### **Храните даты в стандартном формате. Используйте читатели и преобразователи для преобразования формата**

Pis:

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

Yaxşı:

```php
// Модель
protected $dates = ['ordered_at', 'created_at', 'updated_at'];
// Читатель (accessor)
public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// Шаблон
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```

[🔝 Başa qayıt](#Mündəricat)

### **Другие советы и практики**

Не размещайте логику в маршрутах.

Старайтесь не использовать "сырой" PHP в шаблонах Blade.

[🔝 Başa qayıt](#Mündəricat)