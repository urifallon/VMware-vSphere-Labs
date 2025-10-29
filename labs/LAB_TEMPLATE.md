# Lab Template: Quy ước cấu trúc và định dạng

## I. Cấu trúc tổng thể

### 1. Tiêu đề chính
```markdown
# Lab [Số]: [Tên Lab]
```

### 2. Cấu trúc phần chính
```markdown
## I. [Tên phần chính]
## II. [Tên phần chính]
## III. [Tên phần chính]
## IV. [Tên phần chính]
```

### 3. Cấu trúc sub-sections
```markdown
### 1. [Tên sub-section]
### 2. [Tên sub-section]
### 3. [Tên sub-section]
```

### 4. Cấu trúc sub-sub-sections (nếu cần)
```markdown
#### 1.1. [Tên sub-sub-section]
#### 1.2. [Tên sub-sub-section]
```

## II. Quy ước định dạng

### 1. Tiêu đề và số thứ tự
- **Tiêu đề chính:** `# Lab [Số]: [Tên]`
- **Phần chính:** `## I. [Tên phần]`
- **Sub-section:** `### 1. [Tên sub-section]`
- **Sub-sub-section:** `#### 1.1. [Tên sub-sub-section]`

### 2. Hướng dẫn và lưu ý
```markdown
**Hướng dẫn:** [Nội dung hướng dẫn]

**Lưu ý:** [Nội dung lưu ý]

**⚠️ Lưu ý quan trọng:** [Nội dung lưu ý quan trọng]
```

### 3. Bảng cấu hình
```markdown
| Trường | Giá trị |
|--------|---------|
| **Tên trường** | Giá trị cấu hình |
| **Tên trường** | Giá trị cấu hình |
```

### 4. Code blocks
```markdown
**Lệnh kiểm tra:**
```shell
[command here]
```

**Cấu hình:**
```yaml
[configuration here]
```
```

### 5. Hình ảnh
```markdown
![Mô tả hình ảnh](/img/tên-file-hình.png)
```

### 6. Blockquotes
```markdown
> [Nội dung mô tả hoặc lưu ý đặc biệt]
```

## III. Cấu trúc nội dung chuẩn

### 1. Phần mở đầu
- Tiêu đề lab
- Mô tả ngắn gọn về mục tiêu lab
- Yêu cầu tiên quyết (nếu có)

### 2. Phần chuẩn bị
- Yêu cầu phần cứng
- Yêu cầu phần mềm
- Cấu hình mạng
- Các bước chuẩn bị

### 3. Phần thực hiện chính
- Các bước thực hiện chi tiết
- Hình ảnh minh họa
- Bảng cấu hình
- Lệnh cần thiết

### 4. Phần kiểm tra và xác nhận
- Kết quả mong đợi
- Cách kiểm tra
- Xử lý sự cố

### 5. Phần kết thúc
- Tóm tắt
- Lưu ý quan trọng
- Bước tiếp theo

## IV. Quy ước đặt tên

### 1. File hình ảnh
- Format: `[lab-số]-[phần]-[bước].png`
- Ví dụ: `lab1a-buoc1.png`, `lab1b-iii-buoc2.png`

### 2. Thư mục
- Format: `[số]-[tên-lab]`
- Ví dụ: `01-vCenter-Basics`, `02-Storage-vSAN`

### 3. File markdown
- Format: `[tên-lab].md` hoặc `[tên-lab]-[phần].md`
- Ví dụ: `lab1a.md`, `lab1b-sub.md`

## V. Template mẫu

```markdown
# Lab [Số]: [Tên Lab]

## I. [Tên phần chính]

### 1. [Tên sub-section]

**Bước 1:** [Mô tả bước]

![Mô tả hình ảnh](/img/lab-số-buoc1.png)

**Hướng dẫn:** [Hướng dẫn chi tiết]

**Cấu hình:**
| Trường | Giá trị |
|--------|---------|
| **Tên trường** | Giá trị |

→ `Next`

**Lưu ý:** [Lưu ý quan trọng]

### 2. [Tên sub-section tiếp theo]

**Bước 2:** [Mô tả bước]

**Lệnh kiểm tra:**
```shell
[command here]
```

**⚠️ Lưu ý quan trọng:** [Lưu ý đặc biệt]

## II. [Phần chính tiếp theo]

### 1. [Sub-section]

**Quy trình hoạt động:**
1. [Bước 1]
2. [Bước 2]
3. [Bước 3]

## III. Kiểm tra và xác nhận

### Kết quả mong đợi

Sau khi hoàn thành các bước trên, bạn có thể:

1. **[Mục tiêu 1]:** Mô tả
2. **[Mục tiêu 2]:** Mô tả
3. **[Mục tiêu 3]:** Mô tả

### Xử lý sự cố

Nếu gặp lỗi, hãy kiểm tra:

1. **[Vấn đề 1]:** Giải pháp
2. **[Vấn đề 2]:** Giải pháp
3. **[Vấn đề 3]:** Giải pháp

## IV. Lưu ý quan trọng

- **Bảo mật:** [Lưu ý bảo mật]
- **Backup:** [Lưu ý backup]
- **Monitoring:** [Lưu ý monitoring]
- **Documentation:** [Lưu ý documentation]
```

## VI. Checklist tạo lab

### Trước khi viết
- [ ] Xác định mục tiêu lab rõ ràng
- [ ] Chuẩn bị đầy đủ hình ảnh minh họa
- [ ] Lên kế hoạch cấu trúc nội dung
- [ ] Xác định các bước thực hiện chi tiết

### Trong khi viết
- [ ] Sử dụng đúng cấu trúc tiêu đề
- [ ] Thêm hình ảnh minh họa cho mỗi bước
- [ ] Tạo bảng cấu hình rõ ràng
- [ ] Thêm hướng dẫn chi tiết
- [ ] Highlight các lưu ý quan trọng

### Sau khi viết
- [ ] Kiểm tra lại cấu trúc
- [ ] Kiểm tra link hình ảnh
- [ ] Kiểm tra bảng cấu hình
- [ ] Kiểm tra lỗi chính tả
- [ ] Test lại các bước thực hiện

## VII. Ví dụ tham khảo

Xem file `labs/01-vCenter-Basics/lab1a/lab1a.md` để tham khảo cấu trúc và định dạng chuẩn.
