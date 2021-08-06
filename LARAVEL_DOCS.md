<p align="center">
<img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel">
</p>

# 1. Phân quyền (Policy + Gate)
## 1.1. Migration
### Cấu trúc Database
Chúng ta sẽ cần các bảng: users, roles, role_user, permissions, permission_role

```php
Schema::create('users', function (Blueprint $table) {
  $table->bigIncrements('id');
  $table->string('name');
  $table->string('email')->unique();
  $table->timestamp('email_verified_at')->nullable();
  $table->string('password');
  $table->rememberToken();
  $table->timestamps();
});

Schema::create('roles', function (Blueprint $table) {
  $table->increments('id');
  $table->string('name');
  $table->string('display_name');
});

Schema::create('permissions', function (Blueprint $table) {
  $table->increments('id');
  $table->string('display_name');
});

Schema::create('role_user', function (Blueprint $table) {
  $table->unsignedInteger('user_id');
  $table->unsignedInteger('role_id');
  $table->foreign('user_id')->references('id')->on('users')->onDelete('cascade');
  $table->foreign('role_id')->references('id')->on('roles')->onDelete('cascade');
});

Schema::create('permission_role', function (Blueprint $table) {
  $table->unsignedInteger('permission_id');
  $table->unsignedInteger('role_id');
  $table->foreign('permission_id')->references('id')->on('permissions')->onDelete('cascade');
  $table->foreign('role_id')->references('id')->on('roles')->onDelete('cascade');
});
```

## 1.2. Models
### Role model

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

### Permission model

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

### User model

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

## Lưu trữ (Storage)


## Session



## Eloquent ORM (query trong laravel)