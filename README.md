# BÁO CÁO THIẾT KẾ HỆ THỐNG CHATBOT TƯ VẤN THÔNG TIN TRƯỜNG PTIT

## Nhóm thực hiện
1. **Nguyễn Thị Trang** - Mã SV: 20210001 - Module Quản lý người dùng và xác thực
2. **Lê Văn Trọng** - Mã SV: 20210002 - Module Xử lý ngôn ngữ tự nhiên (NLP) và trả lời
3. **Phạm Văn Thuân** - Mã SV: 20210003 - Module Quản lý cơ sở tri thức và dữ liệu

---

# PHẦN 1: THIẾT KẾ KIẾN TRÚC HỆ THỐNG (CHUNG)

## 1.1 Tổng quan hệ thống

Hệ thống thương mại điện tử được xây dựng theo kiến trúc Client-Server 3 tầng (3-tier architecture):

- **Tầng Presentation (Client)**: Giao diện người dùng, xây dựng bằng React/Vue.js
- **Tầng Business Logic (Server)**: Xử lý nghiệp vụ, xây dựng bằng Node.js/Express hoặc Java Spring Boot
- **Tầng Data**: Cơ sở dữ liệu MySQL/PostgreSQL

## 1.2 Sơ đồ kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT LAYER                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Web    │  │  Mobile  │  │  Admin   │  │   API    │   │
│  │   App    │  │   App    │  │  Panel   │  │  Client  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
└─────────────────────────────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Load Balancer │
                    └────────┬────────┘
                             │
┌─────────────────────────────────────────────────────────────┐
│                     SERVER LAYER                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                  API Gateway                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                             │                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Auth    │  │  User    │  │ Product  │  │  Order   │  │
│  │ Service  │  │ Service  │  │ Service  │  │ Service  │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
                             │
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  User    │  │ Product  │  │  Order   │  │  Cache   │  │
│  │    DB    │  │    DB    │  │    DB    │  │  (Redis) │  │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 1.3 Công nghệ sử dụng

### Frontend (Client)
- Framework: React.js / Vue.js
- State Management: Redux / Vuex
- UI Library: Material-UI / Ant Design
- HTTP Client: Axios

### Backend (Server)
- Platform: Node.js với Express.js
- Authentication: JWT (JSON Web Token)
- ORM: Sequelize / TypeORM
- API Documentation: Swagger

### Database
- Primary DB: MySQL / PostgreSQL
- Cache: Redis
- File Storage: AWS S3 / Local Storage

## 1.4 Phân chia module

| Module | Người phụ trách | Chức năng chính |
|--------|----------------|-----------------|
| Quản lý người dùng | Nguyễn Thị Trang | Đăng ký, đăng nhập, quản lý profile, phân quyền |
| Quản lý sản phẩm | Lê Văn Trọng | CRUD sản phẩm, danh mục, tìm kiếm, lọc |
| Quản lý đơn hàng | Phạm Văn Thuân | Giỏ hàng, đặt hàng, thanh toán, theo dõi đơn |

---

# PHẦN 2: THIẾT KẾ CHI TIẾT

## 2.1 THIẾT KẾ CHI TIẾT - NGUYỄN THỊ TRANG (MÃ SV: 20210001)
### Module: Quản lý người dùng

### 2.1.1 Thiết kế CSDL

#### Bảng Users
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    phone VARCHAR(20),
    address TEXT,
    role_id INT,
    status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES roles(role_id)
);
```

#### Bảng Roles
```sql
CREATE TABLE roles (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Bảng Permissions
```sql
CREATE TABLE permissions (
    permission_id INT PRIMARY KEY AUTO_INCREMENT,
    permission_name VARCHAR(100) UNIQUE NOT NULL,
    resource VARCHAR(50),
    action VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Bảng Role_Permissions
```sql
CREATE TABLE role_permissions (
    role_id INT,
    permission_id INT,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(role_id),
    FOREIGN KEY (permission_id) REFERENCES permissions(permission_id)
);
```

### 2.1.2 Thiết kế lớp thực thể

```java
// Entity: User
public class User {
    private int userId;
    private String username;
    private String email;
    private String passwordHash;
    private String fullName;
    private String phone;
    private String address;
    private Role role;
    private UserStatus status;
    private Date createdAt;
    private Date updatedAt;

    // Constructors, getters, setters
}

// Entity: Role
public class Role {
    private int roleId;
    private String roleName;
    private String description;
    private List<Permission> permissions;
    private Date createdAt;

    // Constructors, getters, setters
}

// Entity: Permission
public class Permission {
    private int permissionId;
    private String permissionName;
    private String resource;
    private String action;
    private Date createdAt;

    // Constructors, getters, setters
}

// Enum: UserStatus
public enum UserStatus {
    ACTIVE, INACTIVE, BANNED
}
```

### 2.1.3 Chức năng 1: ĐĂNG KÝ NGƯỜI DÙNG

#### Thiết kế giao diện

**Client Side - Form đăng ký:**
```
┌──────────────────────────────────────┐
│          ĐĂNG KÝ TÀI KHOẢN          │
├──────────────────────────────────────┤
│                                      │
│  Tên đăng nhập: [_______________]    │
│                                      │
│  Email:         [_______________]    │
│                                      │
│  Mật khẩu:      [_______________]    │
│                                      │
│  Xác nhận MK:   [_______________]    │
│                                      │
│  Họ tên:        [_______________]    │
│                                      │
│  Số điện thoại: [_______________]    │
│                                      │
│  [✓] Đồng ý điều khoản sử dụng      │
│                                      │
│     [Đăng ký]    [Hủy]              │
│                                      │
└──────────────────────────────────────┘
```

**Server Side - Admin Panel:**
```
┌────────────────────────────────────────────┐
│         QUẢN LÝ ĐĂNG KÝ MỚI              │
├────────────────────────────────────────────┤
│ Danh sách tài khoản chờ duyệt:           │
│                                            │
│ ┌──┬──────────┬─────────┬──────────────┐ │
│ │ID│ Username │  Email  │   Hành động  │ │
│ ├──┼──────────┼─────────┼──────────────┤ │
│ │1 │ user01   │ a@b.com │ [Duyệt][Từ chối]│ │
│ │2 │ user02   │ c@d.com │ [Duyệt][Từ chối]│ │
│ └──┴──────────┴─────────┴──────────────┘ │
└────────────────────────────────────────────┘
```

#### Biểu đồ lớp chi tiết

```
┌─────────────────────────┐
│   RegisterController    │
├─────────────────────────┤
│ - userService: IUserSvc │
│ - validator: Validator  │
├─────────────────────────┤
│ + register(): Response  │
│ + validateInput(): bool │
└─────────────────────────┘
            │
            │ uses
            ▼
┌─────────────────────────┐
│   UserService           │
├─────────────────────────┤
│ - userRepo: IUserRepo   │
│ - emailService: IEmail  │
│ - encoder: IEncoder     │
├─────────────────────────┤
│ + createUser(): User    │
│ + checkExists(): bool   │
│ + hashPassword(): string│
│ + sendWelcomeEmail()    │
└─────────────────────────┘
            │
            │ uses
            ▼
┌─────────────────────────┐
│   UserRepository        │
├─────────────────────────┤
│ - db: DatabaseConnection│
├─────────────────────────┤
│ + save(user: User): bool│
│ + findByEmail(): User   │
│ + findByUsername(): User│
└─────────────────────────┘
```

**Lý do thiết kế:**
- **Controller**: Tách biệt logic xử lý request/response với business logic
- **Service**: Chứa business logic, đảm bảo Single Responsibility Principle
- **Repository**: Tách biệt data access layer, dễ thay đổi database sau này
- **Interface**: Sử dụng interface để đảm bảo Dependency Inversion Principle

#### Biểu đồ hoạt động

```
┌─────────┐
│  Start  │
└────┬────┘
     │
     ▼
┌──────────────────┐
│ Nhập thông tin   │
│    đăng ký       │
└────────┬─────────┘
         │
         ▼
    ◇─────────◇
   ╱           ╲
  ╱   Validate  ╲     Không hợp lệ
 ◇   thông tin   ◇─────────────┐
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Hợp lệ              │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱  Username   ╲   Tồn tại   │
 ◇   tồn tại?    ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Không               │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱    Email    ╲   Tồn tại   │
 ◇   tồn tại?    ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Không               │
         ▼                     │
┌──────────────────┐           │
│  Mã hóa mật khẩu │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
┌──────────────────┐           │
│  Lưu vào CSDL    │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
┌──────────────────┐           │
│ Gửi email xác   │            │
│     nhận         │           │
└────────┬─────────┘           │
         │                     │
         ▼                     ▼
┌──────────────────┐    ┌──────────────┐
│ Thông báo thành  │    │ Hiển thị lỗi │
│      công        │    └──────────────┘
└────────┬─────────┘
         │
         ▼
    ┌─────────┐
    │   End   │
    └─────────┘
```

#### Biểu đồ tuần tự

```
Client          Controller      UserService      UserRepo        EmailService     Database
  │                 │                │               │                │               │
  │──Register────▶│                │               │                │               │
  │  Request       │                │               │                │               │
  │                 │                │               │                │               │
  │                 │──validate()──▶│               │                │               │
  │                 │                │               │                │               │
  │                 │                │──checkExists─▶│               │               │
  │                 │                │  (username)   │               │               │
  │                 │                │               │                │               │
  │                 │                │               │──SELECT────────────────────▶│
  │                 │                │               │   FROM users                 │
  │                 │                │               │◀───────────null──────────────│
  │                 │                │               │                │               │
  │                 │                │◀──not exists──│               │               │
  │                 │                │               │                │               │
  │                 │                │──checkExists─▶│               │               │
  │                 │                │   (email)     │               │               │
  │                 │                │               │                │               │
  │                 │                │               │──SELECT────────────────────▶│
  │                 │                │               │   FROM users                 │
  │                 │                │               │◀───────────null──────────────│
  │                 │                │               │                │               │
  │                 │                │◀──not exists──│               │               │
  │                 │                │               │                │               │
  │                 │──hashPassword─▶│               │                │               │
  │                 │◀──hashedPwd────│               │                │               │
  │                 │                │               │                │               │
  │                 │──createUser───▶│               │                │               │
  │                 │                │──save(user)──▶│                │               │
  │                 │                │               │──INSERT INTO──────────────▶│
  │                 │                │               │     users                    │
  │                 │                │               │◀──────user_id───────────────│
  │                 │                │◀──success─────│                │               │
  │                 │                │               │                │               │
  │                 │                │──sendEmail───────────────────▶│               │
  │                 │                │               │                │──send()────▶│
  │                 │                │               │                │◀────OK──────│
  │                 │                │◀──────────────email sent──────│               │
  │                 │                │               │                │               │
  │                 │◀──User created─│               │                │               │
  │                 │                │               │                │               │
  │◀──Response──────│               │               │                │               │
  │   (success)     │                │               │                │               │
```

### 2.1.4 Chức năng 2: ĐĂNG NHẬP

#### Thiết kế giao diện

**Client Side - Form đăng nhập:**
```
┌──────────────────────────────────────┐
│            ĐĂNG NHẬP                 │
├──────────────────────────────────────┤
│                                      │
│  Email/Username: [_______________]   │
│                                      │
│  Mật khẩu:       [_______________]   │
│                                      │
│  [✓] Ghi nhớ đăng nhập              │
│                                      │
│     [Đăng nhập]    [Quên MK?]       │
│                                      │
│  ─────────── HOẶC ──────────         │
│                                      │
│  [Đăng nhập với Google]              │
│  [Đăng nhập với Facebook]            │
│                                      │
└──────────────────────────────────────┘
```

**Server Side - Session Management:**
```
┌────────────────────────────────────────────┐
│        QUẢN LÝ PHIÊN ĐĂNG NHẬP           │
├────────────────────────────────────────────┤
│ Sessions hiện tại:                        │
│                                            │
│ ┌──┬──────────┬─────────┬──────────────┐ │
│ │ID│ Username │   IP    │   Thời gian  │ │
│ ├──┼──────────┼─────────┼──────────────┤ │
│ │1 │ admin    │127.0.0.1│ 10:30:25     │ │
│ │2 │ user01   │192.168.1│ 11:45:12     │ │
│ └──┴──────────┴─────────┴──────────────┘ │
│                                            │
│ [Terminate All Sessions] [Refresh]        │
└────────────────────────────────────────────┘
```

#### Biểu đồ lớp chi tiết

```
┌─────────────────────────┐
│    LoginController      │
├─────────────────────────┤
│ - authService: IAuthSvc │
│ - tokenService: IToken  │
├─────────────────────────┤
│ + login(): Response     │
│ + logout(): Response    │
│ + refreshToken(): Token │
└─────────────────────────┘
            │
            │ uses
            ▼
┌─────────────────────────┐
│   AuthService           │
├─────────────────────────┤
│ - userRepo: IUserRepo   │
│ - tokenGen: ITokenGen   │
│ - encoder: IEncoder     │
├─────────────────────────┤
│ + authenticate(): User  │
│ + verifyPassword(): bool│
│ + generateTokens(): Map │
│ + createSession(): void │
└─────────────────────────┘
            │
            │ uses
            ▼
┌─────────────────────────┐
│   SessionRepository     │
├─────────────────────────┤
│ - cache: RedisClient    │
├─────────────────────────┤
│ + createSession(): void │
│ + getSession(): Session │
│ + deleteSession(): void │
└─────────────────────────┘
```

#### Biểu đồ hoạt động

```
┌─────────┐
│  Start  │
└────┬────┘
     │
     ▼
┌──────────────────┐
│ Nhập username/   │
│ email & password │
└────────┬─────────┘
         │
         ▼
    ◇─────────◇
   ╱           ╲
  ╱   Validate  ╲     Không hợp lệ
 ◇   thông tin   ◇─────────────┐
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Hợp lệ              │
         ▼                     │
┌──────────────────┐           │
│ Tìm user trong   │           │
│      CSDL        │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱    User     ╲  Không có   │
 ◇   tồn tại?    ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Có                  │
         ▼                     │
┌──────────────────┐           │
│ Verify password  │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱  Password   ╲    Sai       │
 ◇   đúng?       ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Đúng                │
         ▼                     │
┌──────────────────┐           │
│ Generate JWT     │           │
│     tokens       │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
┌──────────────────┐           │
│ Create session   │           │
└────────┬─────────┘           │
         │                     │
         ▼                     ▼
┌──────────────────┐    ┌──────────────┐
│ Return tokens    │    │ Return error │
│ & user info      │    │   message    │
└────────┬─────────┘    └──────────────┘
         │
         ▼
    ┌─────────┐
    │   End   │
    └─────────┘
```

#### Biểu đồ tuần tự

```
Client          Controller      AuthService      UserRepo        SessionRepo      Database
  │                 │                │               │                │               │
  │──Login────────▶│                │               │                │               │
  │  Request       │                │               │                │               │
  │                 │                │               │                │               │
  │                 │──validate()──▶│               │                │               │
  │                 │                │               │                │               │
  │                 │                │──findUser────▶│               │               │
  │                 │                │               │──SELECT────────────────────▶│
  │                 │                │               │   FROM users                 │
  │                 │                │               │◀────────user data───────────│
  │                 │                │◀───user───────│               │               │
  │                 │                │               │                │               │
  │                 │──verifyPwd────▶│               │                │               │
  │                 │◀───true────────│               │                │               │
  │                 │                │               │                │               │
  │                 │──generateJWT──▶│               │                │               │
  │                 │◀──tokens──────│               │                │               │
  │                 │                │               │                │               │
  │                 │──createSession─▶──────────────▶──────────────▶│               │
  │                 │                │               │                │──SAVE──────▶│
  │                 │                │               │                │   (Redis)    │
  │                 │                │               │                │◀────OK──────│
  │                 │◀──sessionId───────────────────◀──────────────◀│               │
  │                 │                │               │                │               │
  │◀──Response──────│               │               │                │               │
  │  (tokens)      │                │               │                │               │
```

---

## 2.2 THIẾT KẾ CHI TIẾT - LÊ VĂN TRỌNG (MÃ SV: 20210002)
### Module: Quản lý sản phẩm

### 2.2.1 Thiết kế CSDL

#### Bảng Products
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_code VARCHAR(50) UNIQUE NOT NULL,
    product_name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    discount_price DECIMAL(10, 2),
    quantity INT DEFAULT 0,
    category_id INT,
    brand_id INT,
    status ENUM('active', 'inactive', 'out_of_stock') DEFAULT 'active',
    views INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id),
    FOREIGN KEY (brand_id) REFERENCES brands(brand_id)
);
```

#### Bảng Categories
```sql
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(100) NOT NULL,
    parent_id INT,
    slug VARCHAR(100) UNIQUE,
    description TEXT,
    image_url VARCHAR(500),
    sort_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_id) REFERENCES categories(category_id)
);
```

#### Bảng Product_Images
```sql
CREATE TABLE product_images (
    image_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    image_url VARCHAR(500) NOT NULL,
    alt_text VARCHAR(200),
    is_primary BOOLEAN DEFAULT FALSE,
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE
);
```

#### Bảng Product_Attributes
```sql
CREATE TABLE product_attributes (
    attribute_id INT PRIMARY KEY AUTO_INCREMENT,
    product_id INT,
    attribute_name VARCHAR(100),
    attribute_value VARCHAR(200),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE
);
```

### 2.2.2 Thiết kế lớp thực thể

```java
// Entity: Product
public class Product {
    private int productId;
    private String productCode;
    private String productName;
    private String description;
    private BigDecimal price;
    private BigDecimal discountPrice;
    private int quantity;
    private Category category;
    private Brand brand;
    private ProductStatus status;
    private List<ProductImage> images;
    private List<ProductAttribute> attributes;
    private int views;
    private Date createdAt;
    private Date updatedAt;

    // Methods
    public boolean isInStock() {
        return quantity > 0 && status == ProductStatus.ACTIVE;
    }

    public BigDecimal getFinalPrice() {
        return discountPrice != null ? discountPrice : price;
    }

    // Constructors, getters, setters
}

// Entity: Category
public class Category {
    private int categoryId;
    private String categoryName;
    private Category parentCategory;
    private String slug;
    private String description;
    private String imageUrl;
    private int sortOrder;
    private boolean isActive;
    private List<Product> products;
    private Date createdAt;

    // Constructors, getters, setters
}

// Entity: ProductImage
public class ProductImage {
    private int imageId;
    private int productId;
    private String imageUrl;
    private String altText;
    private boolean isPrimary;
    private int sortOrder;
    private Date createdAt;

    // Constructors, getters, setters
}

// Enum: ProductStatus
public enum ProductStatus {
    ACTIVE, INACTIVE, OUT_OF_STOCK
}
```

### 2.2.3 Chức năng 1: THÊM SẢN PHẨM MỚI

#### Thiết kế giao diện

**Server Side - Admin Panel:**
```
┌───────────────────────────────────────────────┐
│             THÊM SẢN PHẨM MỚI                │
├───────────────────────────────────────────────┤
│                                               │
│ Mã sản phẩm:  [_______________] *            │
│                                               │
│ Tên sản phẩm: [_________________________] *  │
│                                               │
│ Danh mục:     [▼ Chọn danh mục        ] *    │
│                                               │
│ Thương hiệu:  [▼ Chọn thương hiệu     ]      │
│                                               │
│ Giá gốc:      [_______________] VNĐ *        │
│                                               │
│ Giá khuyến mãi: [_______________] VNĐ        │
│                                               │
│ Số lượng:     [_______________] *            │
│                                               │
│ Mô tả chi tiết:                              │
│ ┌───────────────────────────────────────┐    │
│ │                                       │    │
│ │                                       │    │
│ └───────────────────────────────────────┘    │
│                                               │
│ Hình ảnh:     [Choose Files] (Max: 5)        │
│ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐         │
│ │    │ │    │ │    │ │    │ │    │         │
│ └────┘ └────┘ └────┘ └────┘ └────┘         │
│                                               │
│ Thuộc tính sản phẩm:                         │
│ [+ Thêm thuộc tính]                          │
│ ┌─────────────────┬─────────────────┐        │
│ │ Tên thuộc tính  │  Giá trị        │        │
│ ├─────────────────┼─────────────────┤        │
│ │ Màu sắc        │ Đen, Trắng, Xanh│        │
│ │ Kích thước     │ S, M, L, XL     │        │
│ └─────────────────┴─────────────────┘        │
│                                               │
│    [Lưu sản phẩm]    [Hủy]                  │
│                                               │
└───────────────────────────────────────────────┘
```

**Client Side - Product Display:**
```
┌──────────────────────────────────────┐
│         CHI TIẾT SẢN PHẨM           │
├──────────────────────────────────────┤
│ ┌──────────┐                         │
│ │          │  Tên sản phẩm           │
│ │  Hình    │  Giá: 500,000 VNĐ       │
│ │   ảnh    │  ⭐⭐⭐⭐⭐ (4.5/5)        │
│ │          │                         │
│ └──────────┘  Số lượng: [1] [-][+]   │
│                                      │
│ [Thêm vào giỏ] [Mua ngay]          │
│                                      │
│ Mô tả sản phẩm:                     │
│ ....................................  │
│                                      │
└──────────────────────────────────────┘
```

#### Biểu đồ lớp chi tiết

```
┌──────────────────────────┐
│   ProductController      │
├──────────────────────────┤
│ - productService: IProdSvc│
│ - validator: Validator   │
│ - fileUpload: IFileUpload│
├──────────────────────────┤
│ + createProduct(): Resp  │
│ + updateProduct(): Resp  │
│ + deleteProduct(): Resp  │
│ + getProducts(): List    │
│ + uploadImages(): List   │
└──────────────────────────┘
            │
            │ uses
            ▼
┌──────────────────────────┐
│   ProductService         │
├──────────────────────────┤
│ - productRepo: IProdRepo │
│ - categoryRepo: ICatRepo │
│ - imageService: IImageSvc│
│ - cacheService: ICache   │
├──────────────────────────┤
│ + addProduct(): Product  │
│ + validateSKU(): bool    │
│ + processImages(): void  │
│ + updateStock(): void    │
│ + calculatePrice(): BigDec│
└──────────────────────────┘
            │
            │ uses
            ▼
┌──────────────────────────┐
│   ProductRepository      │
├──────────────────────────┤
│ - db: DatabaseConnection │
├──────────────────────────┤
│ + save(prod: Product): int│
│ + update(prod: Product)  │
│ + findById(id: int): Prod│
│ + findBySKU(sku: String) │
│ + findByCategory(): List │
└──────────────────────────┘
```

**Lý do thiết kế:**
- **Controller**: Xử lý HTTP requests, validation đầu vào và upload files
- **Service**: Chứa business logic phức tạp như tính giá, xử lý hình ảnh, cache
- **Repository**: Quản lý truy xuất dữ liệu, tối ưu query
- **Separation of Concerns**: Mỗi layer có trách nhiệm riêng biệt

#### Biểu đồ hoạt động

```
┌─────────┐
│  Start  │
└────┬────┘
     │
     ▼
┌──────────────────┐
│ Nhập thông tin   │
│   sản phẩm       │
└────────┬─────────┘
         │
         ▼
    ◇─────────◇
   ╱           ╲
  ╱  Validate   ╲     Không hợp lệ
 ◇  thông tin    ◇─────────────┐
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Hợp lệ              │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱   Mã SP     ╲   Đã có     │
 ◇   tồn tại?    ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ Chưa có             │
         ▼                     │
┌──────────────────┐           │
│ Upload hình ảnh  │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
    ◇─────────◇                │
   ╱           ╲               │
  ╱   Upload    ╲    Lỗi      │
 ◇   thành công? ◇────────────┤
  ╲             ╱              │
   ╲           ╱               │
    ◇─────────◇                │
         │ OK                  │
         ▼                     │
┌──────────────────┐           │
│ Resize & optimize│           │
│     images       │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
┌──────────────────┐           │
│ Lưu thông tin SP │           │
│   vào database   │           │
└────────┬─────────┘           │
         │                     │
         ▼                     │
┌──────────────────┐           │
│ Clear cache      │           │
└────────┬─────────┘           │
         │                     │
         ▼                     ▼
┌──────────────────┐    ┌──────────────┐
│ Thông báo thành  │    │ Hiển thị lỗi │
│      công        │    └──────────────┘
└────────┬─────────┘
         │
         ▼
    ┌─────────┐
    │   End   │
    └─────────┘
```

#### Biểu đồ tuần tự

```
Admin           Controller      ProductService    ProductRepo      ImageService     Database
  │                 │                 │                │                │               │
  │──AddProduct───▶│                 │                │                │               │
  │   Request      │                 │                │                │               │
  │                │                 │                │                │               │
  │                │──validate()────▶│                │                │               │
  │                │                 │                │                │               │
  │                │                 │──checkSKU─────▶│                │               │
  │                │                 │                │──SELECT──────────────────────▶│
  │                │                 │                │   FROM products              │
  │                │                 │                │◀────────null─────────────────│
  │                │                 │◀──not exists───│                │               │
  │                │                 │                │                │               │
  │                │──uploadImages──▶│                │                │               │
  │                │                 │──processImg──────────────────▶│               │
  │                │                 │                │                │──upload────▶│
  │                │                 │                │                │   (S3/Local) │
  │                │                 │                │                │◀───URLs─────│
  │                │                 │◀──────────────image URLs──────│               │
  │                │                 │                │                │               │
  │                │──createProduct─▶│                │                │               │
  │                │                 │──save─────────▶│                │               │
  │                │                 │                │──INSERT INTO─────────────────▶│
  │                │                 │                │   products                   │
  │                │                 │                │◀────product_id───────────────│
  │                │                 │                │                │               │
  │                │                 │                │──INSERT INTO─────────────────▶│
  │                │                 │                │ product_images               │
  │                │                 │                │◀──────OK─────────────────────│
  │                │                 │◀──product_id───│                │               │
  │                │                 │                │                │               │
  │                │                 │──clearCache()─▶│                │               │
  │                │                 │                │                │               │
  │                │◀──Product created│                │                │               │
  │                │                 │                │                │               │
  │◀──Response──────│                │                │                │               │
  │   (success)    │                 │                │                │               │
```

### 2.2.4 Chức năng 2: TÌM KIẾM VÀ LỌC SẢN PHẨM

#### Thiết kế giao diện

**Client Side - Search & Filter:**
```
┌──────────────────────────────────────┐
│      TÌM KIẾM SẢN PHẨM              │
├──────────────────────────────────────┤
│ [🔍 Nhập từ khóa...        ] [Tìm]   │
│                                      │
│ ┌─────────────┐ ┌──────────────────┐│
│ │  BỘ LỌC     │ │   KẾT QUẢ       ││
│ ├─────────────┤ │                  ││
│ │Danh mục:    │ │ Tìm thấy: 25 SP  ││
│ │[✓] Điện tử  │ │                  ││
│ │[ ] Thời trang│ │ Sắp xếp: [▼Giá] ││
│ │[ ] Gia dụng │ │                  ││
│ │             │ │ ┌────┐ ┌────┐   ││
│ │Giá:         │ │ │ SP1│ │ SP2│   ││
│ │Min: [____]  │ │ └────┘ └────┘   ││
│ │Max: [____]  │ │ ┌────┐ ┌────┐   ││
│ │             │ │ │ SP3│ │ SP4│   ││
│ │Thương hiệu: │ │ └────┘ └────┘   ││
│ │[✓] Samsung  │ │                  ││
│ │[ ] Apple    │ │ [1][2][3]...[10] ││
│ └─────────────┘ └──────────────────┘│
└──────────────────────────────────────┘
```

#### Biểu đồ lớp chi tiết

```
┌──────────────────────────┐
│   SearchController       │
├──────────────────────────┤
│ - searchService: ISearch │
│ - filterBuilder: IFilter │
├──────────────────────────┤
│ + search(): SearchResult │
│ + filter(): List<Product>│
│ + suggest(): List<String>│
└──────────────────────────┘
            │
            │ uses
            ▼
┌──────────────────────────┐
│   SearchService          │
├──────────────────────────┤
│ - productRepo: IProdRepo │
│ - elasticClient: IElastic│
│ - cacheService: ICache   │
├──────────────────────────┤
│ + searchProducts(): List │
│ + buildQuery(): Query    │
│ + applyFilters(): List   │
│ + rankResults(): List    │
│ + indexProduct(): void   │
└──────────────────────────┘
```

#### Biểu đồ hoạt động & tuần tự (Tương tự cấu trúc như trên)

---

## 2.3 THIẾT KẾ CHI TIẾT - PHẠM VĂN THUÂN (MÃ SV: 20210003)
### Module: Quản lý đơn hàng

### 2.3.1 Thiết kế CSDL

#### Bảng Orders
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    order_code VARCHAR(50) UNIQUE NOT NULL,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    discount_amount DECIMAL(10, 2) DEFAULT 0,
    shipping_fee DECIMAL(10, 2) DEFAULT 0,
    final_amount DECIMAL(10, 2) NOT NULL,
    payment_method ENUM('cod', 'bank_transfer', 'credit_card', 'e_wallet'),
    payment_status ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    order_status ENUM('pending', 'confirmed', 'processing', 'shipping', 'delivered', 'cancelled') DEFAULT 'pending',
    shipping_address TEXT,
    shipping_phone VARCHAR(20),
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

#### Bảng Order_Items
```sql
CREATE TABLE order_items (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    discount_price DECIMAL(10, 2),
    subtotal DECIMAL(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

#### Bảng Cart
```sql
CREATE TABLE cart (
    cart_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_user_product (user_id, product_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

#### Bảng Payment_Transactions
```sql
CREATE TABLE payment_transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    transaction_code VARCHAR(100) UNIQUE,
    payment_method VARCHAR(50),
    amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'success', 'failed', 'cancelled') DEFAULT 'pending',
    gateway_response TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);
```

### 2.3.2 Thiết kế lớp thực thể

```java
// Entity: Order
public class Order {
    private int orderId;
    private String orderCode;
    private User user;
    private BigDecimal totalAmount;
    private BigDecimal discountAmount;
    private BigDecimal shippingFee;
    private BigDecimal finalAmount;
    private PaymentMethod paymentMethod;
    private PaymentStatus paymentStatus;
    private OrderStatus orderStatus;
    private String shippingAddress;
    private String shippingPhone;
    private String notes;
    private List<OrderItem> orderItems;
    private Date createdAt;
    private Date updatedAt;

    // Methods
    public void calculateTotal() {
        this.totalAmount = orderItems.stream()
            .map(OrderItem::getSubtotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        this.finalAmount = totalAmount
            .subtract(discountAmount)
            .add(shippingFee);
    }

    // Constructors, getters, setters
}

// Entity: OrderItem
public class OrderItem {
    private int itemId;
    private int orderId;
    private Product product;
    private int quantity;
    private BigDecimal unitPrice;
    private BigDecimal discountPrice;
    private BigDecimal subtotal;
    private Date createdAt;

    public void calculateSubtotal() {
        BigDecimal price = discountPrice != null ? discountPrice : unitPrice;
        this.subtotal = price.multiply(BigDecimal.valueOf(quantity));
    }

    // Constructors, getters, setters
}

// Entity: Cart
public class Cart {
    private int cartId;
    private User user;
    private Product product;
    private int quantity;
    private Date addedAt;
    private Date updatedAt;

    // Constructors, getters, setters
}

// Enums
public enum OrderStatus {
    PENDING, CONFIRMED, PROCESSING, SHIPPING, DELIVERED, CANCELLED
}

public enum PaymentStatus {
    PENDING, PAID, FAILED, REFUNDED
}

public enum PaymentMethod {
    COD, BANK_TRANSFER, CREDIT_CARD, E_WALLET
}
```

### 2.3.3 Chức năng 1: GIỎ HÀNG VÀ ĐẶT HÀNG

#### Thiết kế giao diện

**Client Side - Shopping Cart:**
```
┌──────────────────────────────────────┐
│            GIỎ HÀNG                  │
├──────────────────────────────────────┤
│                                      │
│ ┌────────────────────────────────┐  │
│ │ [✓] │ Sản phẩm A │ SL: [-][2][+]│ │
│ │     │ 500,000đ   │ = 1,000,000đ│ │
│ │     │            │ [Xóa]       │ │
│ ├────────────────────────────────┤  │
│ │ [✓] │ Sản phẩm B │ SL: [-][1][+]│ │
│ │     │ 300,000đ   │ = 300,000đ  │ │
│ │     │            │ [Xóa]       │ │
│ └────────────────────────────────┘  │
│                                      │
│ Tạm tính:           1,300,000 VNĐ   │
│ Phí vận chuyển:        30,000 VNĐ   │
│ ─────────────────────────────────    │
│ Tổng cộng:          1,330,000 VNĐ   │
│                                      │
│ Mã giảm giá: [___________] [Áp dụng]│
│                                      │
│ [Tiếp tục mua hàng] [Thanh toán]    │
└──────────────────────────────────────┘
```

**Client Side - Checkout:**
```
┌──────────────────────────────────────┐
│          THANH TOÁN                  │
├──────────────────────────────────────┤
│ THÔNG TIN GIAO HÀNG                 │
│                                      │
│ Họ tên:     [___________________]   │
│ Điện thoại: [___________________]   │
│ Địa chỉ:    [___________________]   │
│             [___________________]   │
│                                      │
│ PHƯƠNG THỨC THANH TOÁN              │
│ ( ) Thanh toán khi nhận hàng (COD)  │
│ ( ) Chuyển khoản ngân hàng          │
│ (●) Thẻ tín dụng/Debit              │
│ ( ) Ví điện tử                      │
│                                      │
│ THÔNG TIN ĐƠN HÀNG                  │
│ ┌────────────────────────────┐      │
│ │ 2 sản phẩm    1,300,000đ │      │
│ │ Vận chuyển       30,000đ │      │
│ │ Giảm giá        -50,000đ │      │
│ │ ───────────────────────── │      │
│ │ Tổng:        1,280,000đ  │      │
│ └────────────────────────────┘      │
│                                      │
│ Ghi chú: [___________________]      │
│                                      │
│    [Đặt hàng]    [Quay lại]        │
└──────────────────────────────────────┘
```

#### Biểu đồ lớp chi tiết

```
┌──────────────────────────┐
│   OrderController        │
├──────────────────────────┤
│ - orderService: IOrderSvc│
│ - cartService: ICartSvc  │
│ - paymentSvc: IPaymentSvc│
├──────────────────────────┤
│ + addToCart(): Response  │
│ + updateCart(): Response │
│ + checkout(): Response   │
│ + placeOrder(): Order    │
│ + getOrderStatus(): Status│
└──────────────────────────┘
            │
            │ uses
            ▼
┌──────────────────────────┐
│   OrderService           │
├──────────────────────────┤
│ - orderRepo: IOrderRepo  │
│ - productRepo: IProdRepo │
│ - inventorySvc: IInventory│
│ - emailService: IEmail   │
├──────────────────────────┤
│ + createOrder(): Order   │
│ + validateStock(): bool  │
│ + reserveStock(): bool   │
│ + calculateTotal(): BigDec│
│ + processPayment(): bool │
│ + sendConfirmation(): void│
└──────────────────────────┘
            │
            │ uses
            ▼
┌──────────────────────────┐
│   OrderRepository        │
├──────────────────────────┤
│ - db: DatabaseConnection │
├──────────────────────────┤
│ + save(order: Order): int│
│ + findById(id: int): Order│
│ + findByUser(uid: int):List│
│ + updateStatus(): bool   │
└──────────────────────────┘
```

**Lý do thiết kế:**
- **Controller**: Điều phối luồng đặt hàng từ giỏ hàng đến thanh toán
- **Service**: Xử lý logic phức tạp như kiểm tra tồn kho, tính toán, thanh toán
- **Repository**: Quản lý transaction đảm bảo tính toàn vẹn dữ liệu
- **Integration**: Tích hợp với các service khác (inventory, payment, email)

#### Biểu đồ hoạt động

```
┌─────────┐
│  Start  │
└────┬────┘
     │
     ▼
┌──────────────────┐
│ Xem giỏ hàng     │
└────────┬─────────┘
         │
         ▼
    ◇─────────◇
   ╱           ╲
  ╱  Giỏ hàng   ╲     Rỗng
 ◇   có SP?      ◇────────────┐
  ╲             ╱             │
   ╲           ╱              │
    ◇─────────◇               │
         │ Có                 │
         ▼                    │
┌──────────────────┐          │
│ Nhập thông tin   │          │
│   giao hàng      │          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
    ◇─────────◇               │
   ╱           ╲              │
  ╱  Validate   ╲   Invalid   │
 ◇   thông tin   ◇───────────┤
  ╲             ╱             │
   ╲           ╱              │
    ◇─────────◇               │
         │ Valid              │
         ▼                    │
┌──────────────────┐          │
│ Chọn phương thức │          │
│   thanh toán     │          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
┌──────────────────┐          │
│ Kiểm tra tồn kho │          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
    ◇─────────◇               │
   ╱           ╲              │
  ╱   Đủ hàng   ╲   Không đủ │
 ◇     ?         ◇───────────┤
  ╲             ╱             │
   ╲           ╱              │
    ◇─────────◇               │
         │ Đủ                 │
         ▼                    │
┌──────────────────┐          │
│ Reserve inventory│          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
┌──────────────────┐          │
│ Xử lý thanh toán │          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
    ◇─────────◇               │
   ╱           ╲              │
  ╱  Thanh toán ╲    Lỗi     │
 ◇   thành công? ◇───────────┤
  ╲             ╱             │
   ╲           ╱              │
    ◇─────────◇               │
         │ OK                 │
         ▼                    │
┌──────────────────┐          │
│ Tạo đơn hàng     │          │
└────────┬─────────┘          │
         │                    │
         ▼                    │
┌──────────────────┐          │
│ Gửi email xác    │          │
│     nhận         │          │
└────────┬─────────┘          │
         │                    │
         ▼                    ▼
┌──────────────────┐   ┌──────────────┐
│ Hiển thị mã đơn  │   │ Thông báo lỗi│
└────────┬─────────┘   └──────────────┘
         │
         ▼
    ┌─────────┐
    │   End   │
    └─────────┘
```

#### Biểu đồ tuần tự

```
Client        Controller     OrderService    ProductRepo    PaymentSvc    OrderRepo      Database
  │               │               │              │              │             │              │
  │──Checkout───▶│               │              │              │             │              │
  │               │               │              │              │             │              │
  │               │──getCart()───▶│              │              │             │              │
  │               │◀──cartItems───│              │              │             │              │
  │               │               │              │              │             │              │
  │               │──validateStock▶             │              │             │              │
  │               │               │──checkStock─▶│              │             │              │
  │               │               │              │──SELECT──────────────────────────────▶│
  │               │               │              │   products                            │
  │               │               │              │◀──────────stock levels────────────────│
  │               │               │◀──inStock────│              │             │              │
  │               │               │              │              │             │              │
  │──PlaceOrder──▶│              │              │              │             │              │
  │               │──createOrder─▶│              │              │             │              │
  │               │               │              │              │             │              │
  │               │               │──reserveStock▶             │             │              │
  │               │               │              │──UPDATE──────────────────────────────▶│
  │               │               │              │  quantity                             │
  │               │               │              │◀────────────OK─────────────────────────│
  │               │               │◀──reserved───│              │             │              │
  │               │               │              │              │             │              │
  │               │               │──processPayment────────────▶│            │              │
  │               │               │              │              │──charge()─▶│              │
  │               │               │              │              │◀──success──│              │
  │               │               │◀─────────────payment OK─────│            │              │
  │               │               │              │              │             │              │
  │               │               │──saveOrder──────────────────────────────▶│              │
  │               │               │              │              │             │──INSERT────▶│
  │               │               │              │              │             │   orders    │
  │               │               │              │              │             │◀──order_id──│
  │               │               │◀─────────────order created───────────────│              │
  │               │               │              │              │             │              │
  │               │               │──sendEmail()─▶             │             │              │
  │               │               │              │              │             │              │
  │               │◀──Order#12345─│              │              │             │              │
  │               │               │              │              │             │              │
  │◀──Response────│              │              │              │             │              │
  │  (order info) │               │              │              │             │              │
```

### 2.3.4 Chức năng 2: THEO DÕI ĐƠN HÀNG

#### Thiết kế giao diện

**Client Side - Order Tracking:**
```
┌──────────────────────────────────────┐
│       THEO DÕI ĐƠN HÀNG             │
├──────────────────────────────────────┤
│ Mã đơn: #ORD-2024-001234            │
│ Ngày đặt: 15/01/2024 10:30          │
│                                      │
│ TRẠNG THÁI ĐƠN HÀNG                 │
│                                      │
│ [✓]──[✓]──[✓]──[●]──[ ]──[ ]       │
│  Đặt  Xác  Đóng  Vận  Giao Hoàn    │
│ hàng  nhận  gói  chuyển hàng tất    │
│                                      │
│ Trạng thái hiện tại: Đang vận chuyển│
│ Dự kiến giao: 18/01/2024            │
│                                      │
│ CHI TIẾT ĐƠN HÀNG                   │
│ ┌────────────────────────────────┐  │
│ │ 1. Sản phẩm A x2   1,000,000đ │  │
│ │ 2. Sản phẩm B x1     300,000đ │  │
│ └────────────────────────────────┘  │
│                                      │
│ Địa chỉ giao: 123 ABC, Q1, TP.HCM   │
│ SĐT: 0901234567                     │
│                                      │
│ [Hủy đơn]  [Liên hệ hỗ trợ]        │
└──────────────────────────────────────┘
```

**Server Side - Order Management:**
```
┌───────────────────────────────────────────────┐
│          QUẢN LÝ ĐƠN HÀNG                   │
├───────────────────────────────────────────────┤
│ Bộ lọc: [▼Tất cả] [▼Hôm nay] [Tìm kiếm___] │
│                                               │
│ ┌───┬──────────┬──────────┬────────────────┐│
│ │STT│ Mã đơn   │Khách hàng│ Trạng thái     ││
│ ├───┼──────────┼──────────┼────────────────┤│
│ │ 1 │ORD-1234 │ Nguyễn A │ ⚪ Chờ xác nhận││
│ │ 2 │ORD-1235 │ Trần B   │ 🟡 Đang xử lý  ││
│ │ 3 │ORD-1236 │ Lê C     │ 🟢 Đang giao   ││
│ │ 4 │ORD-1237 │ Phạm D   │ ✅ Hoàn thành  ││
│ └───┴──────────┴──────────┴────────────────┘│
│                                               │
│ [Export Excel] [In báo cáo] [Cập nhật lọc]  │
└───────────────────────────────────────────────┘
```

#### Biểu đồ lớp và các biểu đồ khác (tương tự cấu trúc đã trình bày ở trên)

---

# PHẦN 3: TỔNG KẾT

## 3.1 Tích hợp các module

Hệ thống được thiết kế theo kiến trúc microservices với 3 module chính:
- Module Quản lý người dùng (Trang): Xử lý authentication, authorization
- Module Quản lý sản phẩm (Trọng): Quản lý catalog, inventory
- Module Quản lý đơn hàng (Thuân): Xử lý đặt hàng, thanh toán

Các module giao tiếp qua RESTful API và message queue để đảm bảo loose coupling.

## 3.2 Kế hoạch triển khai

1. **Phase 1**: Xây dựng core services và database
2. **Phase 2**: Phát triển APIs và business logic
3. **Phase 3**: Xây dựng UI/UX cho client và admin
4. **Phase 4**: Testing và optimization
5. **Phase 5**: Deployment và monitoring

## 3.3 Công nghệ và tools

- Version Control: Git/GitHub
- Project Management: Jira/Trello
- CI/CD: Jenkins/GitLab CI
- Container: Docker/Kubernetes
- Monitoring: Prometheus/Grafana

---

**KẾT THÚC BÁO CÁO**
