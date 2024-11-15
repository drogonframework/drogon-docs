# Câu hỏi thường gặp

Đây là danh sách các câu hỏi thường gặp và câu trả lời, với một số giải thích mở rộng.

## Mô hình luồng và các thực tiễn tốt nhất của Drogon là gì?

Drogon chạy trên một nhóm luồng (thread pool), trong đó các luồng máy chủ HTTP cũng như luồng cơ sở dữ liệu được tạo khi `app().run()` được gọi. Nó là một hệ thống dựa trên tác vụ tuần tự. Do đó, bạn nên luôn sử dụng các API bất đồng bộ hoặc coroutine khi có thể. Xem [Hiểu mô hình luồng của Drogon](VI-FAQ-1-Understanding-drogon-threading-model) để biết chi tiết.

