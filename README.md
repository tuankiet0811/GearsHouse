# GearsHouse

GearsHouse là ứng dụng thương mại điện tử cho phép duyệt sản phẩm, quản lý giỏ hàng, thanh toán trực tuyến, theo dõi đơn hàng và quản trị nội dung hệ thống. Dự án được xây dựng trên ASP.NET Core MVC (.NET 9) với Identity, Entity Framework Core và Razor Pages.

## Tổng quan

- Bán hàng trực tuyến cho các sản phẩm linh kiện/gear.
- Danh mục, thương hiệu, khuyến mãi, đánh giá, chat nội bộ, mã giảm giá.
- Giỏ hàng, đặt hàng, xuất hóa đơn PDF, email thông báo.
- Tích hợp cổng thanh toán VNPay và Momo (sandbox).
- Khu vực quản trị (Dashboard) dành cho Admin: sản phẩm, danh mục, thương hiệu, khuyến mãi, đơn hàng, người dùng, báo cáo doanh thu.

## Kiến trúc

- Lớp trình bày: MVC Controllers + Razor Views/Pages, routing cấu hình trong [Program.cs](file:///e:/GearsHouse/GearsHouse/GearsHouse/Program.cs).
- Xác thực & phân quyền: ASP.NET Core Identity với role Admin/User; cookie auth cấu hình trong Program.cs.
- Truy cập dữ liệu: Entity Framework Core qua [ApplicationDbContext](file:///e:/GearsHouse/GearsHouse/GearsHouse/Models/ApplicationDbContext.cs) và các Repository (IProductRepository, ICategoryRepository, IBrandRepository).
- Dịch vụ: EmailService, InvoicePdfGenerator (xuất PDF), VNPayService, MomoService.
- Phiên làm việc: DistributedMemoryCache + Session cho giỏ hàng và trạng thái người dùng.
- Cấu hình: appsettings.json và appsettings.Development.json, Secret Manager cho khóa nhạy cảm.

## Tính năng chính

- Duyệt, tìm kiếm, lọc, sắp xếp sản phẩm theo danh mục/brand/giá.
- Quản lý hình ảnh sản phẩm, thông số kỹ thuật, khuyến mãi.
- Giỏ hàng: thêm/xóa/cập nhật số lượng, áp mã giảm giá.
- Đặt hàng: tạo đơn, thanh toán VNPay/Momo, nhận email xác nhận, xuất hóa đơn PDF.
- Theo dõi đơn hàng của người dùng, xem lịch sử.
- Kênh chat nội bộ giữa người dùng và hệ thống.
- Quản trị: quản lý sản phẩm, danh mục, thương hiệu, khuyến mãi, người dùng & vai trò, báo cáo doanh thu.

## Yêu cầu hệ thống

- .NET SDK 9.0
- SQL Server (LocalDB hoặc SQL Server 2019+)
- Visual Studio 2022 hoặc VS Code (tùy chọn)
- Cấu hình SMTP để gửi email (Gmail hoặc dịch vụ khác)
- Tài khoản sandbox VNPay/Momo (khóa truy cập/secret)

## Công nghệ sử dụng

- Nền tảng: ASP.NET Core MVC (.NET 9), Razor Pages, Identity.
- ORM: Entity Framework Core (SqlServer).
- Thư viện: MailKit/MimeKit (email), QuestPDF/PDFsharp/iText7 (PDF), Newtonsoft.Json.
- Session & Cache: DistributedMemoryCache + Session.
- Công cụ scaffolding: Microsoft.VisualStudio.Web.CodeGeneration.Design.
- Migrations: Thư mục Migrations có sẵn, dùng EF Core Tools.

## Hướng dẫn cài đặt

1) Clone mã nguồn

```bash
git clone <repo_url>
cd GearsHouse/GearsHouse/GearsHouse
```

2) Cấu hình kết nối DB và khóa bí mật

- Sử dụng Secret Manager thay vì commit vào appsettings.json (khuyến nghị):

```bash
# Khởi tạo user-secrets (nếu chưa)
dotnet user-secrets init

# Chuỗi kết nối SQL Server
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=<your_server>;Database=GearsHouse;Trusted_Connection=True;TrustServerCertificate=True"

# Email
dotnet user-secrets set "EmailSettings:FromEmail" "<your_email>"
dotnet user-secrets set "EmailSettings:DisplayName" "GEARSHOUSE"
dotnet user-secrets set "EmailSettings:Password" "<app_password>"
dotnet user-secrets set "EmailSettings:Host" "smtp.gmail.com"
dotnet user-secrets set "EmailSettings:Port" "587"
dotnet user-secrets set "EmailSettings:EnableSSL" "true"

# VNPay
dotnet user-secrets set "VNPay:TmnCode" "<tmn_code>"
dotnet user-secrets set "VNPay:HashSecret" "<hash_secret>"
dotnet user-secrets set "VNPay:BaseUrl" "https://sandbox.vnpayment.vn/paymentv2/vpcpay.html"
dotnet user-secrets set "VNPay:ReturnUrl" "https://localhost:7203/ShoppingCart/VNPayReturn"

# Momo
dotnet user-secrets set "MomoAPI:MomoApiUrl" "https://test-payment.momo.vn/v2/gateway/api/create"
dotnet user-secrets set "MomoAPI:SecretKey" "<secret_key>"
dotnet user-secrets set "MomoAPI:AccessKey" "<access_key>"
dotnet user-secrets set "MomoAPI:ReturnUrl" "https://localhost:7203/ShoppingCart/MomoReturn"
dotnet user-secrets set "MomoAPI:NotifyUrl" "https://localhost:7203/ShoppingCart/MomoNotify"
dotnet user-secrets set "MomoAPI:PartnerCode" "MOMO"
dotnet user-secrets set "MomoAPI:RequestType" "captureWallet"
```

3) Khôi phục và build

```bash
dotnet restore
dotnet build
```

4) Tạo database bằng migrations (nếu chưa có)

```bash
# Cài dotnet-ef nếu chưa
dotnet tool install --global dotnet-ef

# Áp dụng migrations
dotnet ef database update
```

5) Chạy ứng dụng

```bash
dotnet run
# Ứng dụng lắng nghe theo launchSettings: https://localhost:7203; http://localhost:5121
```

## Các endpoint chính

Các route mặc định theo cấu hình MVC truyền thống: `{controller}/{action}/{id?}`. Dưới đây là các endpoint tiêu biểu:

- Home
  - GET `/Home/Index`
  - GET `/Home/FeaturedProducts?mode=best|new`
  - GET `/Home/NextFortySale`
  - GET `/Home/TrendingProducts?count=12`

- Product
  - GET `/Product/Index?categoryId&brandId&keyword`
  - GET `/Product/Display/{id}`
  - GET `/Product/Add` (Admin)
  - POST `/Product/Add` (Admin)
  - GET `/Product/Update/{id}` (Admin)
  - POST `/Product/Update/{id}` (Admin)
  - GET `/Product/Delete/{id}` (Admin)
  - POST `/Product/DeleteConfirmed` (Admin)
  - GET `/Product/Filter?keyword&categoryId&brandId&minPrice&maxPrice&sort`
  - GET `/Product/Suggest?keyword&take=6`

- Categories (Admin)
  - GET `/Categories/Index`
  - GET `/Categories/AddCategory`
  - POST `/Categories/AddCategory`
  - POST `/Categories/DeleteCategoryConfirmed/{id}`

- Brand (Admin)
  - Các endpoint tương tự quản lý thương hiệu (Index/Add/Update/Delete).

- Promotion (Admin)
  - GET `/Promotion/Index`

- Dashboard (Admin)
  - GET `/Dashboard/Dashboard?tab=revenue|product|category|brand|promotion|order|user`
  - GET `/Dashboard/ProductIndex`
  - GET `/Dashboard/CategoryIndex`
  - GET `/Dashboard/BrandIndex`
  - GET `/Dashboard/PromotionIndex`
  - GET `/Dashboard/OrderIndex`
  - GET `/Dashboard/UserIndex`
  - POST `/Dashboard/UpdateUserRole`
  - POST `/Dashboard/DeleteUser`
  - GET `/Dashboard/Revenue`

- ShoppingCart (User đã đăng nhập)
  - GET `/ShoppingCart/AddToCart?productId&quantity`
  - GET `/ShoppingCart/BuyNow?productId&quantity=1`
  - GET `/ShoppingCart/IndexAsync`
  - GET `/ShoppingCart/UpdateQuantity?productId&change`
  - GET `/ShoppingCart/RemoveFromCartAsync?productId`
  - GET `/ShoppingCart/CheckoutFromCart`
  - POST `/ShoppingCart/CreateOrder`
  - GET `/ShoppingCart/Checkout`
  - GET `/ShoppingCart/CheckoutExisting/{id}`
  - POST `/ShoppingCart/Checkout`
  - POST `/ShoppingCart/ApplyCoupon?code&orderId`
  - POST `/ShoppingCart/IncreaseQuantity?productId`
  - POST `/ShoppingCart/DecreaseQuantity?productId`
  - POST `/ShoppingCart/CancelOrder?id`
  - GET `/ShoppingCart/GetCartItemCount` (AllowAnonymous)
  - GET `/ShoppingCart/VNPayReturn`
  - GET `/ShoppingCart/MomoReturn`
  - POST `/ShoppingCart/MomoNotify`

- Order (User/Admin)
  - GET `/Order/GetRecentOrdersJson`
  - GET `/Order/Tracking`
  - GET `/Order/OrderManage` (Admin)

- Chat (User đã đăng nhập)
  - GET `/Chat/MyThread`
  - GET `/Chat/Messages?threadId&afterId`
  - GET `/Chat/GetUnreadCount`
  - POST `/Chat/SEND?threadId&content`

- Identity (mặc định của ASP.NET Identity UI)
  - `/Identity/Account/Login`, `/Identity/Account/Register`, `/Identity/Account/Manage`...

## Bảo mật

- Không commit khóa bí mật: di chuyển EmailSettings, VNPay, Momo, ConnectionStrings sang Secret Manager hoặc biến môi trường.
- Bắt buộc HTTPS: giữ HSTS ở môi trường sản xuất; triển khai reverse proxy/nginx để buộc HTTPS.
- Cookie/Session an toàn: cấu hình cookie `Secure` và `HttpOnly`, thời gian sống hợp lý; hạn chế dữ liệu nhạy cảm trong session.
- CSRF & XSS: sử dụng AntiForgeryToken cho form POST; validate & encode dữ liệu đầu vào/hiển thị.
- Phân quyền chặt chẽ: dùng `[Authorize]` và `[Authorize(Roles="Admin")]` cho endpoint quản trị; không để lộ dữ liệu hệ thống qua JSON/partial view.
- Thanh toán an toàn: xác minh chữ ký trả về từ VNPay/Momo, kiểm tra trạng thái đơn hàng theo server-side, không tin cậy tham số từ client.
- Cơ sở dữ liệu: dùng tham số hóa truy vấn (EF Core mặc định), migration kiểm soát schema, backup/khôi phục định kỳ.

## Cấu hình tham khảo

- Xem [GearsHouse.csproj](file:///e:/GearsHouse/GearsHouse/GearsHouse/GearsHouse.csproj) để biết TargetFramework, gói phụ thuộc.
- Cấu hình môi trường phát triển: [appsettings.Development.json](file:///e:/GearsHouse/GearsHouse/GearsHouse/appsettings.Development.json).
- Điểm khởi tạo ứng dụng và middleware: [Program.cs](file:///e:/GearsHouse/GearsHouse/GearsHouse/Program.cs).

