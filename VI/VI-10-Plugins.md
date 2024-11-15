## Plugin

Plugin được sử dụng để giúp người dùng xây dựng các ứng dụng phức tạp. Trong Drogon, tất cả các plugin được build và cài đặt vào ứng dụng dựa trên tệp cấu hình. Plugin trong Drogon là đơn thể (single-instance), và người dùng có thể triển khai bất kỳ chức năng nào họ muốn với plugin.

Khi Drogon chạy giao diện `run()`, nó sẽ khởi tạo từng plugin một theo tệp cấu hình và gọi giao diện `initAndStart()` của chúng.

### Cấu hình

Cấu hình plugin được thực hiện thông qua tệp cấu hình, ví dụ:

```json
    "plugins": [
        {
            // name: Tên lớp của plugin
            "name": "DataDictionary",
            // dependencies: Danh sách các plugin mà plugin này phụ thuộc vào. Có thể được comment bỏ qua
            "dependencies": [],
            // config: Cấu hình của plugin. Đối tượng JSON này là tham số để khởi tạo plugin.
            // Có thể được comment bỏ qua
            "config": {
            }
        }
    ],
```

Có thể thấy rằng có ba cấu hình cho mỗi plugin:

- `name`: là tên lớp của plugin (bao gồm cả namespace). Framework sẽ tạo một instance plugin dựa trên tên lớp. Nếu mục này được comment bỏ qua, plugin sẽ bị vô hiệu hóa.
- `dependencies`: Là danh sách tên của các plugin khác mà plugin này phụ thuộc vào. Framework tạo và khởi tạo tất cả các plugin theo một thứ tự cụ thể. Ưu tiên tạo và khởi tạo các plugin mà các plugin khác phụ thuộc vào. Khi kết thúc chương trình, các plugin bị đóng và hủy theo thứ tự ngược lại. Xin lưu ý rằng các phụ thuộc vòng tròn (circular dependencies) trong plugin bị cấm. Drogon sẽ báo lỗi và thoát khỏi chương trình nếu nó phát hiện ra một phụ thuộc vòng tròn. Nếu mục này được comment bỏ qua, danh sách các phần phụ thuộc sẽ trống.
- `config`: là đối tượng JSON được sử dụng để khởi tạo plugin, đối tượng được truyền dưới dạng tham số đầu vào cho giao diện `initAndStart()` của plugin. Nếu mục này được comment bỏ qua, đối tượng JSON được truyền cho giao diện `initAndStart` là một đối tượng rỗng;

### Định nghĩa

Plugin do người dùng định nghĩa phải kế thừa từ template lớp `drogon::Plugin`, và tham số template là kiểu plugin, chẳng hạn như định nghĩa sau:

```c++
class DataDictionary : public drogon::Plugin<DataDictionary>
{
public:
    virtual void initAndStart(const Json::Value &config) override;
    virtual void shutdown() override;
    // ...
};
```

Người ta có thể tạo các tệp nguồn của plugin bằng lệnh `drogon_ctl`:

```shell
drogon_ctl create plugin <[namespace::]tên_lớp>
```

### Lấy Instance

Instance plugin được tạo bởi Drogon, và người dùng có thể lấy instance plugin thông qua giao diện sau của Drogon:

```c++
template <typename T> T *getPlugin();
```

Hoặc

```c++
PluginBase *getPlugin(const std::string &name);
```

Rõ ràng, phương thức đầu tiên thuận tiện hơn. Ví dụ: plugin `DataDictionary` được đề cập ở trên có thể được lấy như thế này:

```c++
auto *pluginPtr = app().getPlugin<DataDictionary>();
```

Lưu ý rằng tốt nhất bạn nên lấy plugin sau khi gọi giao diện `run()` của framework, nếu không người ta sẽ nhận được một instance plugin chưa được khởi tạo (điều này không nhất thiết dẫn đến lỗi, chỉ cần đảm bảo sử dụng plugin sau khi khởi tạo là được). Tất nhiên, vì plugin được khởi tạo theo thứ tự phụ thuộc, nên việc lấy instance của plugin khác trong giao diện `initAndStart()` không có vấn đề gì.

### Vòng đời

Tất cả các plugin được khởi tạo trong giao diện `run()` của framework và bị hủy khi ứng dụng thoát. Do đó, vòng đời của plugin gần như giống hệt với ứng dụng, đó là lý do tại sao giao diện `getPlugin()` không cần trả về một con trỏ thông minh.


# Tiếp theo: [Tệp Cấu hình](VI-11-Configuration-File)


