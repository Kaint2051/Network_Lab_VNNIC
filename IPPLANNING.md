# **CHIA IP theo dải 103.95.196.0/22**

   ![Screenshot 2023-04-12 013955](https://user-images.githubusercontent.com/72087963/231258813-08a209f8-3f15-4105-9be9-fd006d717cd8.png)

```Dải IP và yêu cầu của Lab```

Ở đây ta có thể thấy yêu cầu của bài Lab là chia dãy IP trên cho 8 khu chính (vì 1 địa chỉ IPv4 rất đắt và chúng mình đã thử tự chia nhưng vẫn thừa quá nhiều, nên các anh Mentor đã hỗ trợ hay còn gọi là giải thẳng dùm tụi mình luôn :v). Nếu đã học qua các khoá quy hoạch IP "siêu cấp tốc or from zero to negative number =))" thì ắt hẵn ai làm Network cũng biết mình sẽ thường chia IP từ Host lớn đến nhỏ.
   - Đầu tiên, mình gọi là Request 1, đây là host lớn nhất là Văn phòng gồm 200 user. Vì lấy hơn 200 user + dự phòng có thể mở rộng nên tụi mình lấy luôn /24 là 254 cho đủ một mạng nhé. Vậy dải IP của host này là: 103.95.196.0/24

   - Tiếp theo là Request 2 là host Kỹ thuật(hay còn gọi là cán bộ quản trị). Đây là host gồm 32 user + dự phòng nên mình sẽ lấy /26 là 60 là đủ(lúc đầu mình định lấy /27 nhưng subnet này chỉ có 30 user nên sẽ ko đủ cho 32 user). Và ở Request 1 đã lấy hết 1 dải mạng 103.95.196.0/24 nên mình cần + thêm 1bit đẩy qua phần network sẽ thành 103.95.197.0. Vậy là địa chỉ IP của host này là: 103.95.197.0/26

   - Ở Request 3, ta sẽ thấy có 2 request là IP dành cho server(Request 3.1) và IP dành cho quản trị thiết bị(Request 3.2). Vì đây là lab mang tính dự phòng cao nên chúng ta sẽ chia theo 2 request này trước, mỗi host gồm 1 dãy địa chỉ ~ 32, ở đây mình sẽ lấy /27(mình vẫn có thể lấy /28 or /26 nhưng mình thấy IP dành cho 2 request này mà cần nâng cấp trong tương lai thì /27 sẽ đủ [nếu ko đủ thì kệ chứ, người tính sao mà bằng trời tính được =)))]). Hiện tại địa chỉ 103.95.197.0/26 chỉ mới lấy tới 103.95.197.63 của request 2 trên nên mỗi request 3.1 và 3.2 mình sẽ + thêm cho 32 host. Vậy nên địa chỉ IP của 
     + Request 3.1 là: 103.95.197.64/27 
     + Request 3.2 là: 103.95.197.96/27(+ 32bit của /27 sang phần host của Request 3.1)

   - Request 4 sẽ là IP dành cho Routing từ Layer 3(từ Router trở ra) nên ta chỉ cần ~ 16 Ip là đủ. Vì chỉ lấy khoảng 16 Ip nên mình lấy /28(thực tế là 14 Ip) để làm subnet cho host này. Và từ Request 3.2 ta sẽ + thêm 16bit cho phần host thì ta sẽ được dải IP là: 103.95.197.128/28.

   - Ở 2 request cuối là IP dành cho kế toán([Host này mà không chia cuối tháng nhịn đói :v] Request 5.1) và IP dành cho Lãnh đạo(Request 5.2), mỗi request khoảng 10 user nên mình sẽ chia theo /28 luôn cho tiện. Tiếp trên dải IP từ Request 4 là 103.95.197.128 mình sẽ + thêm mỗi Request là 16bit cho phần host.
     + Request 5.1: 103.95.197.144/28
     + Request 5.2: 103.95.197.160/28

   ![image](https://user-images.githubusercontent.com/72087963/231259222-87ab7b57-3d81-4dd5-be1b-6e1e445ca837.png)


`Tuỳ theo cách chia mình có thể chia và lấy IP dự phòng theo nhiều cách khác nhau và không nhất thiết phải giống như cách làm của tụi mình nhé. 😄`

