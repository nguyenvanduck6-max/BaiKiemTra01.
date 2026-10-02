cau 1:
Value Types (Kiểu giá trị) là những kiểu dữ liệu lưu trực tiếp giá trị của biến. Các biến kiểu giá trị thường được lưu trên vùng nhớ Stack. Stack là vùng nhớ có tốc độ truy cập nhanh, được cấp phát và giải phóng tự động khi phương thức kết thúc. Khi gán một biến kiểu giá trị cho biến khác, toàn bộ giá trị của biến sẽ được sao chép sang vùng nhớ mới. Vì vậy, hai biến hoàn toàn độc lập với nhau. Nếu thay đổi giá trị của một biến thì biến còn lại sẽ không bị ảnh hưởng. Một số kiểu dữ liệu thuộc Value Types gồm: int, float, double, decimal, char, bool, struct và enum.
Ví dụ:

int a = 10;
int b = a;

b = 20;

// a = 10
// b = 20

ví dụ trên, khi thay đổi giá trị của b, biến a vẫn giữ nguyên giá trị ban đầu vì hai biến được lưu ở hai vùng nhớ khác nhau.

Ngược lại, Reference Types (Kiểu tham chiếu) không lưu trực tiếp dữ liệu mà chỉ lưu địa chỉ của đối tượng. Đối tượng thực tế được cấp phát trên vùng nhớ Heap, còn biến chỉ lưu địa chỉ để tham chiếu đến đối tượng đó. Khi gán một biến tham chiếu cho biến khác, chỉ có địa chỉ của đối tượng được sao chép, vì vậy cả hai biến đều cùng trỏ đến một đối tượng trên Heap. Nếu thay đổi dữ liệu thông qua một biến thì dữ liệu của biến còn lại cũng sẽ thay đổi theo. Các kiểu dữ liệu thuộc Reference Types gồm có class, object, string, array, delegate và interface.

Ví dụ:
class Student
{
    public string Name;
}

Student s1 = new Student();
s1.Name = "Nam";

Student s2 = s1;
s2.Name = "An";

// s1.Name = "An"
// s2.Name = "An"

Câu 2:

Đối với thuộc tính sử dụng set, giá trị của thuộc tính có thể được thay đổi bất cứ lúc nào trong suốt quá trình chương trình thực thi. Sau khi tạo đối tượng, lập trình viên vẫn có thể sửa giá trị của thuộc tính nhiều lần nếu cần.

Ví dụ:

public class Student
{
    public string Name { get; set; }
}

Student s = new Student();
s.Name = "Nguyễn Văn A";
s.Name = "Trần Văn B";

Trong ví dụ trên, thuộc tính Name có thể được thay đổi nhiều lần vì sử dụng set.

Trong khi đó, thuộc tính sử dụng init chỉ cho phép gán giá trị trong quá trình khởi tạo đối tượng. Sau khi đối tượng đã được tạo xong, mọi thao tác thay đổi giá trị của thuộc tính sẽ bị trình biên dịch báo lỗi.

Ví dụ:

public class Product
{
    public string ProductId { get; init; }
}

Product p = new Product
{
    ProductId = "SP001"
};

// p.ProductId = "SP002"; // Báo lỗi

Việc sử dụng init giúp bảo vệ những dữ liệu quan trọng không bị thay đổi ngoài ý muốn, từ đó làm tăng tính an toàn và ổn định của chương trình.

Trong thực tế, init thường được sử dụng đối với những thông tin không nên thay đổi sau khi đối tượng được tạo như mã sinh viên, mã sản phẩm, mã nhân viên, số CCCD, biển số xe, mã đơn hàng hoặc ngày sinh. Những dữ liệu này thường chỉ được xác định một lần và không nên chỉnh sửa trong quá trình sử dụng hệ thống.

Như vậy, điểm khác nhau cơ bản là set cho phép thay đổi dữ liệu nhiều lần sau khi tạo đối tượng, còn init chỉ cho phép gán dữ liệu duy nhất trong lúc khởi tạo đối tượng. Việc sử dụng init giúp xây dựng các đối tượng bất biến và tăng tính bảo mật cũng như tính toàn vẹn của dữ liệu.

Câu 3:
Trong C#, để thực hiện tính đa hình, người ta sử dụng hai từ khóa là virtual và override.

Từ khóa virtual được khai báo trong lớp cha để cho phép các lớp con có thể ghi đè phương thức đó. Phương thức virtual đã có sẵn phần cài đặt mặc định. Nếu lớp con không ghi đè thì chương trình sẽ sử dụng phần cài đặt của lớp cha.

Ví dụ:

class Animal
{
    public virtual void Speak()
    {
        Console.WriteLine("Animal speaks");
    }
}

Từ khóa override được khai báo trong lớp con để thay thế hoàn toàn phần cài đặt của phương thức virtual trong lớp cha. Khi đối tượng của lớp con được gọi thông qua biến của lớp cha, chương trình sẽ tự động thực hiện phương thức đã được ghi đè trong lớp con.

Ví dụ:

class Dog : Animal
{
    public override void Speak()
    {
        Console.WriteLine("Dog barks");
    }
}

Khi thực hiện:

Animal animal = new Dog();
animal.Speak();

Kết quả hiển thị sẽ là:

Dog barks

Điều này chứng tỏ mặc dù biến có kiểu dữ liệu là Animal, nhưng khi đối tượng thực tế là Dog thì phương thức Speak() của lớp Dog sẽ được gọi. Đây chính là cơ chế đa hình trong C#.

Có thể hiểu đơn giản rằng virtual là phương thức ở lớp cha cho phép ghi đè, còn override là phương thức ở lớp con dùng để định nghĩa lại cách hoạt động của phương thức đó. Hai từ khóa này kết hợp với nhau giúp chương trình linh hoạt hơn, dễ mở rộng hơn và giảm sự phụ thuộc giữa các lớp.

Câu 4:
Khi sử dụng toán tử new, chương trình chỉ tạo ra các đối tượng mới và cấp phát bộ nhớ cho các thành viên không phải static. Trong khi đó, các thành viên static đã được tạo ngay khi lớp được nạp vào bộ nhớ và không phụ thuộc vào bất kỳ đối tượng nào.

Ví dụ:

class Student
{
    public static int Count = 0;
}

Cách truy cập đúng là:

Student.Count++;

Nếu truy cập thông qua đối tượng:

Student s = new Student();
s.Count++;

thì trong các phiên bản C# hiện đại, trình biên dịch sẽ phát sinh cảnh báo vì cách viết này dễ gây hiểu nhầm rằng Count thuộc về đối tượng s, trong khi thực tế Count thuộc về lớp Student.

Một ứng dụng thực tế của static là đếm số lượng đối tượng được tạo ra.

Ví dụ:

class Student
{
    public static int TotalStudent = 0;

    public Student()
    {
        TotalStudent++;
    }
}

Student s1 = new Student();
Student s2 = new Student();

Console.WriteLine(Student.TotalStudent);

Kết quả sẽ là:2
