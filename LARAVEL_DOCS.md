<p align="center">
<img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel">
</p>

## 1. Phân quyền (Policy + Gate)
### Migration
Chúng ta sẽ cần các bảng: users, roles, role_user, permissions, permission_role

```php
Schema::create('users', function (Blueprint $table) 
{
  $table->bigIncrements('id');
  $table->string('name');
  $table->string('email')->unique();
  $table->timestamp('email_verified_at')->nullable();
  $table->string('password');
  $table->rememberToken();
  $table->timestamps();
});

Schema::create('roles', function (Blueprint $table) 
{
  $table->increments('id');
  $table->string('name');
  $table->string('display_name');
});

Schema::create('permissions', function (Blueprint $table) 
{
  $table->increments('id');
  $table->string('display_name');
});

Schema::create('role_user', function (Blueprint $table) 
{
  $table->unsignedInteger('user_id');
  $table->unsignedInteger('role_id');
  $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
  $table->foreign('role_id')->references('id')->on('roles')->onDelete('cascade');
});

Schema::create('permission_role', function (Blueprint $table) 
{
  $table->unsignedInteger('permission_id');
  $table->unsignedInteger('role_id');
  $table->foreign('permission_id')->references('id')->on('permissions')->onDelete('cascade');
  $table->foreign('role_id')->references('id')->on('roles')->onDelete('cascade');
});
```
### Models
`Role model`

```php
class Role extends Model
{
  protected $table = 'roles';
  public $timestamps = false;

  public function permissions()
  {
    return $this->belongsToMany('App\Models\Permission');
  }

  public function user()
  {
    return $this->belongsToMany('App\Models\User');
  }
}
```

`Permission model`

```php
class Permission extends Model
{
  protected $table = 'permissions';
  public $timestamps = false;

  public function role()
  {
    return $this->belongsToMany('App\Models\Role');
  }
}
```

`User model`

```php
class User extends Model
{
  protected $table = 'users';

  public function role()
  {
    return $this->belongsToMany('App\Models\Role');
  }
}
```

## Lưu trữ hình ảnh (Upload Image Storage)
Để upload được ảnh trong Laravel thì ở form mình thêm 1 attribute `enctype="multipart/form-data"`. Ở bên `Controller` ta làm như sau:

```php
$pic = $request->picture;
$picName = 'picture-'.time().'.'.$pic->extension();
$pic->move(public_path('picture'), $picName);

User::create([
  ...,
  'picture' => $picName;
  ...,
]);
```

Giải thích:
  - Tạo biến *picName* để lưu tên theo dạng `[tên]-[thời_gian_tạo].[định_dạng_file_ảnh]`
  - Lưu ảnh vừa tạo vào thư mục `public` với folder *picture* và biến *picName*

## Session
Là một phiên làm việc, duy trì cho đến khi tắt browser. Tuy nhiên, Laravel định nghĩa cho 1 session tồn tại trong vòng 2 tiếng và vẫn hoạt động cho dù đã tắt browser. Bạn có thể settings theo ý muốn bằng cách truy cập vào `config/session` và settings:

```php
'lifetime' => env('SESSION_LIFETIME', 120),
'expire_on_close' => false,
```

- set cho **SESSION_LIFETIME** thành bao nhiêu tuỳ ý, **120** là 120 phút.
- set cho **expire_on_close** thành `true` nếu bạn muốn session mất đi khi tắt browser.

Chúng ta có thể tạo chức năng đăng nhập. Tuy nhiên Laravel đã tạo ra giúp bạn tính năng này và support vô cùng đầy đủ là `Authentication`, tìm hiểu [tại đây](https://laravel.com/docs/master/authentication). Hãy cùng code 1 tính năng đăng nhập và đăng xuất sử dụng session đơn giản bằng email và số điện thoại.

```html
<form action="{{ route('login') }}" method="POST">
  <input type="hidden" name="_token" value="{{ csrf_token() }}">
  <input type="email" name="email">
  <input type="password" name="phone">
  <input type="submit" value="Sign in">
</form>
```

```php
session_start();
public function login(Request $request) 
{
  $user = User::where('email', $request->email)->first();
  session()->put('email', $user->email);
  session()->put('name', $user->name);

  if ($user->phone === $request->phone) {
    return redirect()->route('home');
  } else {
    return back()->with('error', 'Email hoặc số điện thoại không đúng');
  }
}

public function logout()
{
  if (session()->has('email')) {
    session()->forget(['email', 'name']);
  }
  return redirect()->route('signin');
}
```
Để khởi tạo được session thì trước tiên trong php chúng ta phải khai báo `session_start()`. Tạo method đăng nhập như sau:
- Tạo biến `user` và query lấy data từ `$request->email`
- Tạo session `email` từ `$user->email`
- Tạo session `name` từ `$user->name`
- Check điều kiện nếu data từ `$user->phone` trùng khớp với `$request->phone` nhập vào thì chuyển hướng đến `route('home')` còn không thì quay lại và trả 1 *flash session* là Email hoặc số điện thoại không đúng.

Đã có đăng nhập thì phải có đăng xuất:
- Check điều kiện xem session email có tồn tại không bằng cách `session()->has('email')`. Nếu tồn tại thì ta sử dụng method `forget()` có sẵn để xoá session (method này cho phép truyền vào 1 session hoặc 1 array session).
- Chuyển hướng về trang có form đăng nhập.

**Lưu ý**: Có 2 loại session là: session và flash session. Sự khác nhau là session tồn tại từ lúc tạo cho đến khi expire còn flash session chỉ tồn tại duy nhất 1 lần cho đến khi trang load lại thì nó sẽ mất đi. 


## Eloquent ORM (query trong laravel)