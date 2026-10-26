## THỰC HÀNH: TRIỂN KHAI VÀ ĐÁNH GIÁ WAF (MODSECURITY + NGINX)

### Cấu hình tệp `docker-compose.yml`

```yaml
services:
  web:
    image: bkimminich/juice-shop
    ports: ["3000:3000"]
  waf:
    image: owasp/modsecurity-crs:nginx
    ports: ["8080:8080"]
    environment:
      BACKEND: http://web:3000
      MODSEC_RULE_ENGINE: "On"
      BLOCKING_PARANOIA: "1"
```
### Thực hành đăng nhập

#### 1. Không có WAF
Mở localhost:3000 và đăng nhập với:
- Email: `' OR 1=1 --`
- Mật khẩu: `123456`

Kết quả: Đăng nhập thành công với role Admin
<img width="1897" height="917" alt="image" src="https://github.com/user-attachments/assets/20460506-71cd-4325-89c1-cceb12a48c4b" />


#### 2. Có WAF
Mở localhost:8080, đăng nhập y hệt:
- Email: `' OR 1=1 --`
- Mật khẩu: `123456`

Kết quả: bị chặn 403 Forbidden
<img width="1900" height="912" alt="image" src="https://github.com/user-attachments/assets/90369492-a7d4-4e80-ba81-6a59c3d78621" />


#### 3. Đăng nhập đúng
Qua localhost:8080 với:
- Email: `admin@juice-sh.op`
- Mật khẩu: `admin123`

Kết quả: Đăng nhập bình thường
<img width="1897" height="912" alt="image" src="https://github.com/user-attachments/assets/90d657f9-43ed-4cbb-9a7d-016176ba6edf" />


#### 4. Xem lý do
Nhập `docker compose logs waf` vào terminal

Kết quả: Thấy luật 942100 (SQL injection)  
"messages":[{"message":"SQL Injection Attack Detected via libinjection",
"details":{
  "match":"detected SQLi using libinjection.",
  "ruleId":"942100",
  ...
  "data":"Matched Data: s&1c found within ARGS:json.email: ' OR 1=1 --",
  "severity":"2",
  "tags":["application-multi", "attack-sqli", "OWASP_CRS/ATTACK-SQLI"]
}}
