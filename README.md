# Bài 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

## 1. Mục tiêu

Bài thực hành nhằm tìm hiểu và áp dụng hai cơ chế `git reset` và `git revert` để xử lý commit chứa mã nguồn bị lỗi.

Qua bài thực hành, em thực hiện được:

* Sử dụng `git reset` để đưa lịch sử local lùi lại một commit.
* Giữ lại nội dung thay đổi trong Working Directory dưới dạng `Modified`.
* Sử dụng `git revert` để đảo ngược một commit bằng cách tạo một commit mới.
* Phân biệt được `reset` và `revert`.
* Hiểu cách sử dụng hai cơ chế này khi làm việc cá nhân và khi làm việc nhóm trên branch chung.

---

# 2. Chuẩn bị

Repository được tạo riêng để thực hành bài tập.

File được sử dụng để mô phỏng mã nguồn:

```text
app.txt
```

Lịch sử commit ban đầu:

```text
Initial application
        ↓
Add payment feature
        ↓
Add buggy payment logic
```

Trong đó `Add buggy payment logic` là commit chứa mã nguồn bị lỗi.

---

# 3. Trường hợp 1 - Git Reset

## 3.1. Tạo commit chứa mã nguồn lỗi

Sau khi tạo repository, em thực hiện các commit:

```bash
git add app.txt
git commit -m "Initial application"
```

Sau đó thêm chức năng thanh toán:

```bash
git add app.txt
git commit -m "Add payment feature"
```

Tiếp theo, em thêm nội dung lỗi vào file `app.txt` và tạo commit:

```bash
git add app.txt
git commit -m "Add buggy payment logic"
```

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Kết quả có dạng:

```text
xxxxxxx Add buggy payment logic
xxxxxxx Add payment feature
xxxxxxx Initial application
```

Lúc này commit `Add buggy payment logic` đang nằm trên đầu nhánh.

---

## 3.2. Kiểm tra trạng thái trước khi reset

Sử dụng:

```bash
git status
```

Kết quả:

```text
On branch master
nothing to commit, working tree clean
```

Điều này cho thấy toàn bộ thay đổi đã được commit và Working Directory đang sạch.

---

## 3.3. Thực hiện Git Reset

Vì commit lỗi mới chỉ tồn tại ở local và chưa được chia sẻ lên branch chung, em sử dụng:

```bash
git reset HEAD~1
```

Lệnh trên sử dụng chế độ `--mixed` mặc định.

Có thể viết đầy đủ:

```bash
git reset --mixed HEAD~1
```

Sau khi thực hiện, HEAD được đưa lùi lại một commit.

Lịch sử từ:

```text
Initial application
        ↓
Add payment feature
        ↓
Add buggy payment logic
```

trở thành:

```text
Initial application
        ↓
Add payment feature
```

Tuy nhiên, nội dung của commit lỗi vẫn được giữ lại trong file `app.txt`.

---

## 3.4. Kiểm tra sau Reset

Kiểm tra trạng thái:

```bash
git status
```

Kết quả mong đợi:

```text
Changes not staged for commit:
  modified:   app.txt
```

Điều này chứng minh:

* HEAD đã lùi lại một commit.
* Commit `Add buggy payment logic` không còn nằm trên lịch sử hiện tại.
* Nội dung của commit lỗi không bị mất.
* File `app.txt` đang ở trạng thái `Modified`.
* Thay đổi chưa được đưa vào Staging Area.

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Kết quả:

```text
xxxxxxx Add payment feature
xxxxxxx Initial application
```

---

## 3.5. Lưu ý

Trong trường hợp này không sử dụng:

```bash
git reset --hard HEAD~1
```

vì `--hard` có thể xóa luôn các thay đổi trong Working Directory.

Đề bài yêu cầu giữ lại mã nguồn hiện hành ở trạng thái `Modified`, vì vậy sử dụng:

```bash
git reset HEAD~1
```

là phù hợp.

---

# 4. Trường hợp 2 - Git Revert

## 4.1. Tạo lại commit lỗi

Sau khi thực hiện Reset, nội dung lỗi vẫn còn trong `app.txt`.

Để thực hành trường hợp Revert, em tạo lại commit lỗi:

```bash
git add app.txt
git commit -m "Add buggy payment logic"
```

Kiểm tra:

```bash
git log --oneline -n 5
```

Kết quả:

```text
xxxxxxx Add buggy payment logic
xxxxxxx Add payment feature
xxxxxxx Initial application
```

Trong trường hợp này giả sử commit lỗi đã được push lên remote và các thành viên khác trong nhóm đã có commit này.

---

## 4.2. Tại sao không sử dụng Reset?

Nếu commit đã được push lên branch chung, việc sử dụng:

```bash
git reset
```

để xóa commit khỏi lịch sử sẽ làm thay đổi lịch sử của branch.

Điều này có thể gây khó khăn cho các thành viên khác khi đồng bộ repository.

Vì vậy, trong trường hợp commit đã được công khai, sử dụng `git revert` là an toàn hơn.

---

## 4.3. Thực hiện Git Revert

Sử dụng:

```bash
git revert HEAD
```

Git sẽ tạo một commit mới có tác dụng đảo ngược các thay đổi của commit lỗi.

Lịch sử từ:

```text
Initial application
        ↓
Add payment feature
        ↓
Add buggy payment logic
```

trở thành:

```text
Initial application
        ↓
Add payment feature
        ↓
Add buggy payment logic
        ↓
Revert "Add buggy payment logic"
```

Commit lỗi cũ vẫn tồn tại trong lịch sử.

---

## 4.4. Kiểm tra sau Revert

Kiểm tra lịch sử:

```bash
git log --oneline -n 5
```

Kết quả mong đợi:

```text
xxxxxxx Revert "Add buggy payment logic"
xxxxxxx Add buggy payment logic
xxxxxxx Add payment feature
xxxxxxx Initial application
```

Kết quả trên chứng minh Git đã tạo một commit mới để đảo ngược commit lỗi.

Kiểm tra trạng thái:

```bash
git status
```

Kết quả:

```text
On branch master
nothing to commit, working tree clean
```

---

# 5. So sánh Git Reset và Git Revert

| Tiêu chí                    | Git Reset                            | Git Revert                  |
| --------------------------- | ------------------------------------ | --------------------------- |
| Tạo commit mới              | Không                                | Có                          |
| Commit cũ còn trong lịch sử | Không còn trên lịch sử hiện tại      | Vẫn còn                     |
| Thay đổi lịch sử            | Có                                   | Không                       |
| Đảo ngược thay đổi          | Có thể đưa HEAD về trước commit      | Tạo commit mới để đảo ngược |
| Phù hợp với commit local    | Có                                   | Có                          |
| Phù hợp với commit đã push  | Không nên                            | Có                          |
| Phù hợp với branch chung    | Không nên nếu commit đã được chia sẻ | Phù hợp                     |
| An toàn khi làm việc nhóm   | Thấp nếu đã push                     | Cao                         |
| Lệnh thực hiện              | `git reset HEAD~1`                   | `git revert HEAD`           |

---

# 6. Sử dụng Reset khi làm việc cá nhân

Khi làm việc cá nhân, nếu commit lỗi chỉ tồn tại trên máy local và chưa được push lên remote, có thể sử dụng:

```bash
git reset HEAD~1
```

Ví dụ:

```text
A → B → C
```

Trong đó `C` là commit lỗi.

Sau Reset:

```text
A → B
```

HEAD quay về `B`.

Nếu sử dụng `git reset HEAD~1` (`--mixed`), nội dung của `C` vẫn được giữ lại trong Working Directory dưới dạng `Modified`.

Điều này phù hợp khi lập trình viên muốn sửa lại code và commit lại theo cách khác.

---

# 7. Sử dụng Revert khi làm việc nhóm

Khi commit đã được push lên remote hoặc đã được các thành viên khác lấy về, không nên tự ý viết lại lịch sử bằng Reset.

Ví dụ branch chung đang có:

```text
A → B → C
```

và `C` là commit lỗi.

Thay vì xóa `C`, sử dụng:

```bash
git revert C
```

Git tạo thêm commit:

```text
A → B → C → D
```

Trong đó:

```text
D = Revert C
```

Commit `C` vẫn tồn tại trong lịch sử nhưng các thay đổi của `C` được đảo ngược bởi `D`.

Cách này giúp lịch sử Git rõ ràng và các thành viên khác có thể tiếp tục đồng bộ branch mà không phải xử lý việc lịch sử bị viết lại.

---

# 8. Kết luận

Qua bài thực hành, em đã thực hiện hai phương pháp xử lý commit lỗi.

### Git Reset

Sử dụng:

```bash
git reset HEAD~1
```

Kết quả:

* HEAD lùi lại một commit.
* Commit lỗi không còn nằm trên lịch sử hiện tại.
* Nội dung thay đổi vẫn được giữ lại.
* File chuyển sang trạng thái `Modified`.

Reset phù hợp với các commit chỉ tồn tại ở local, đặc biệt khi làm việc cá nhân và chưa push lên remote.

### Git Revert

Sử dụng:

```bash
git revert HEAD
```

Kết quả:

* Commit lỗi cũ vẫn được giữ trong lịch sử.
* Git tạo thêm một commit mới.
* Commit mới có nội dung đảo ngược thay đổi của commit lỗi.
* Lịch sử không bị viết lại.

Revert phù hợp khi commit đã được push lên remote hoặc nằm trên branch chung có nhiều thành viên cùng làm việc.

---

# 9. Minh chứng thực hiện

## 9.1. Minh chứng Trường hợp 1 - Reset

Sử dụng:

```bash
git status
git log --oneline -n 5
```

Ảnh chụp cần thể hiện:

```text
modified: app.txt
```

và commit lỗi không còn nằm trên đầu lịch sử.

---

## 9.2. Minh chứng Trường hợp 2 - Revert

Sử dụng:

```bash
git log --oneline -n 5
```

Ảnh chụp cần thể hiện:

```text
xxxxxxx Revert "Add buggy payment logic"
xxxxxxx Add buggy payment logic
xxxxxxx Add payment feature
xxxxxxx Initial application
```

Đây là minh chứng cho việc đã tạo thành công commit `Revert`.

---

# 10. Tổng kết

`git reset` và `git revert` đều có thể được sử dụng để xử lý commit sai, nhưng cách hoạt động khác nhau:

```text
RESET:

A → B → C
        ↓
     reset
        ↓
A → B

Revert:

A → B → C
        ↓
      revert
        ↓
A → B → C → D

D = Revert C
```

Vì vậy:

* **Commit local, chưa push:** có thể sử dụng `reset`.
* **Commit đã push hoặc branch chung:** nên sử dụng `revert`.
* **Không sử dụng `reset --hard` trong bài này** vì yêu cầu phải giữ lại mã nguồn hiện hành dưới dạng `Modified`.
