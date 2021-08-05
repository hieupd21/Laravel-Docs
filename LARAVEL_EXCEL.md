<p align="center">
<img src="https://user-images.githubusercontent.com/7728097/67977317-26178100-fc18-11e9-943e-c5302ca32622.png" width="400" height="82" alt="Laravel Excel logo">
</p>

## ✨ Tính năng
- **Dễ dàng xuất dữ liệu sang Excel.**
- **Xuất các truy vấn với tính năng phân khúc tự động để có hiệu suất tốt hơn.**
- **Nhập data và sheet sang Eloquent với chức năng đọc chunk và chèn hàng loạt.**
- **Sử dụng HTML trong dạng Blade template và xuất bảng đó sang Excel.**

## 🔧 Cài đặt
Yêu cầu gói này trong `composer.json` của dự án Laravel của bạn. Thao tác này sẽ tải xuống gói và **PhpS Spreadsheet**

  composer require maatwebsite/excel

Thêm **ServiceProvider** trong `config/app.php`

  Maatwebsite\Excel\ExcelServiceProvider::class,

Thêm **Facade** trong `config/app.php`

  'Excel' => Maatwebsite\Excel\Facades\Excel::class,

Để xuất bản cấu hình, hãy chạy lệnh:

  php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider" --tag=config

Thao tác này sẽ tạo một tệp cấu hình mới có tên `config/excel.php`

## 🚀 Export
### Quick Start
Tạo 1 **Export Class** trên `app/Exports`
Bạn có thể thực hiện việc này bằng cách sử dụng lệnh `make:export`

  php artisan make:export UsersExport --model=User

Trong class này sẽ sử lý trả về 1 query đơn giản ở function `collection()` :

```php
  public function collection()
  {
    return User::all();
  }
```

Và để sử dụng được function `collection()` này ta phải implements 1 interface là **FromCollection**
Khai báo 1 route ở `route.php` để trỏ hướng export

```php
  Route::get('export', 'UsersController@export');
```

Ở **UserController** bạn có thể gọi export như sau:

```php
  public function export() 
  {
    return Excel::download(new UsersExport, 'users.xlsx');
  }
```

📄 Tìm users.xlsx của bạn trong thư mục Download của bạn. 
Tuy nhiên lúc này file export ra chỉ có tất cả data trong model `User` chứ không hề được customize theo ý của bạn, cho nên chúng ta sẽ tạo thêm 2 function để phục vụ cho điều này !!!
Chúng ta tạo function `map()` để chọn ra các trường mong muốn:

```php
  public function map($user): array
  {
    return [
      $user->id,
      $user->name,
      date('d-m-Y', strtotime($user->created_at)),
    ];
  }
```

Và để sử dụng được function `map()` thì ta phải implement 1 interface là **WithMapping**
Để hiển thị tên của các cột thì chúng ta sử dụng function `headings()` :

```php
  public function headings(): array
  {
    return [
      'Mã người dùng',
      'Tên người dùng',
      'Ngày tạo',
    ];
  }
```

Và để sử dụng được function `map()` thì ta phải implement 1 interface là **WithHeadings**

### Style
Bạn có thể tạo style của riêng mình trong excel.

```php
  public function registerEvents(): array
  {
    return [
      AfterSheet::class => function (AfterSheet $event) {
        $event->sheet->getStyle('A1:C1')->applyFromArray([
          'font' => [
            'bold' => true,
          ],
          'border' => [
            'outline' => [
              'borderStyle' => \PhpOffice\PhpSpreadsheet\Style\BorderStyle::class,
              'color' => ['argb' => 'FFFF0000'],
            ],
          ],
        ]);
      }
    ];
  }
```

Và để sử dụng được function `registerEvents()` thì ta phải implement 1 interface là **WithEvents**

### Exportables
Laravel Excel cũng cung cấp `Maatwebsite\Excel\Concerns\Exportable`, và chúng ta thêm vào class như sau:

```php
  use Exportable;
```

Thông thường chúng ta sẽ customize câu truy vấn nên sẽ phải có 1 constuctor nhận các tham số từ controller:

```php
  public function __construct($year)
  {
    $this->year = $year;
  }

  public function query()
  {
    return Invoice::query()->whereYear('created_at', $this->year);
  }
```

Ở **UserController** ta gọi function `export` có tham số như ở ``__constructor``:

```php
  public function export($year) 
  {
    return Excel::download(new UsersExport($year), 'users.xlsx');
  }
```

Ở `route.php` thì:

```php
  Route::get('export/{year}', 'UsersController@export');
```

Và để sử dụng được function `query()` thì ta phải implement 1 interface là **FromQuery**

### ShouldAutoSize
Khi export ra thì kích thước ở các row sẽ không đều và đẹp vì thế hay implement interface **ShouldAutoSize** để kích thước tự điều chỉnh và đẹp hơn.