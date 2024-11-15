## Thông tin về Brotli

Drogon hỗ trợ các tệp được nén tĩnh Brotli ngay khi xuất xưởng nếu nó tìm thấy tài nguyên (asset) được nén Brotli tương ứng bên cạnh tài nguyên.

Vì vậy, chẳng hạn, Drogon sẽ tìm kiếm `/đường_dẫn/đến/tài_nguyên.js.br` cho yêu cầu `/đường_dẫn/đến/tài_nguyên.js`.

Nó thực hiện bằng cách đặt `br_static` thành `true` theo mặc định trong tệp `config.json`.

Nếu bạn muốn nén động với Brotli, bạn sẽ phải đặt `use_brotli` thành `true` trong `config.json`.

Người dùng không có ý định sử dụng Brotli tĩnh, có thể muốn loại bỏ "kiểm tra anh chị em" (sibling check) bổ sung của Brotli bằng cách đặt `br_static` thành `false` trong `config.json`.

# 16 [Coroutine](VI-16-Coroutines)


