## Các bước triển khai cài đặt CA

**Chuẩn bị:**
-	Tải các file binary fabric-ca-server, fabric-ca-client từ Hyperledger fabric về máy.
-	Các thành phần cần cài đặt:
```
    • TLS CA :  CA cung cấp định danh cho TLS.
    • Organization CA: CA cung cấp định danh cho các thành viên của một tổ chức.
    • fabric-ca-Client: Dùng để tương tác với các CA.
```
### I. Cài đặt TLS CA
1.	Tạo một folder để chứa các file: trong hướng dẫn này sẽ là tls_ca
2.	Copy file fabric-ca-server vào thư mục mới tạo
3.	Khởi tạo máy chủ TLS CA
- Trong folder **tls_ca** bật terminal và chạy dòng lệnh: 
`./fabric-ca-server init -b <ADMIN_USER>:<ADMIN_PWD>`
trong đó `ADMIN_USER` là tài khoản của admin cho tls ca, `ADMIN_PWD` là mật khẩu
Ví dụ:
`./fabric-ca-server init -b admintls:admintlspw`
Sau khi chạy lệnh này xong thì trong folder tls_ca sẽ tự động sinh ra các file và folder cần thiết cho quá trình cài đặt.
4. Sửa đổi thông tin ở file fabric-ca-server-conflig.yaml được sinh ra ở bước trên trong tls_ca theo ý muốn.
    ```
    Tối thiểu nên đổi các trường sau:
    - port: Nhập cổng mà bạn muốn sử dụng cho máy chủ này. Trong hướng dẫn này sử dựng port 6666
    - tls.enabled: Đặt là true hoặc false, mặc định là false. Đay là thiết lập bật tắt sử dụng TLS, trong dự án thực tế thì cần thiết bật, trong hướng dẫn này để là true
    - ca.name : Đặt tên cho TLS CA của bạn
    - crs.hosts: Cập nhật tham số này bao gồm tên máy chủ này và địa chỉ ip nơi máy chủ này đang chạy, nếu nó khác với những thông tin được ghi trong tệp. Tên máy chủ sẽ được sử dụng để chỉ định tên thay thế khi máy chủ tạo chứng chỉ TLS tự ký tls-cert.pem khi khởi chạy máy chủ trong bước tiếp theo.
    - sign.profiles.ca - Vì đây là TLS CA sẽ không cấp chứng chỉ CA nên phần profiles.ca có thể xóa. Khối sign.profiles chỉ nên chứa mình mục tls.
    - operations.listenAddress: Thay đổi số port để dự phòng trong trường hợp có một chương trình khác chạy trên máy này và sử dụng port giống với số port được thiết lập bên trên.

    ```
    ***Note:*** Trong trường hợp bạn sửa đổi bất kỳ giá trị nào trong khối crs của file fabric-ca-server-conflig.yaml thì bạn cần xóa tls_ca/ca-cert.pem file và toàn bộ tls_ca/msp. Các file này sẽ được tự động sinh lại trong bước tiếp theo.

5. Khởi động máy chủ TLS CA
Chạy câu lệnh `./fabric-ca-server start` để khởi chạy máy chủ TLS CA

6. Đăng ký identity cho admin TLS CA
- Sử dụng fabric-ca-client để đăng ký.
- Tạo một folder để lưu trữ chương trình cho fabric-ca-client, trong hướng dẫn này tên ca_client được sử dụng và cấu trúc của nó như sau:
    ```
    ca_client
    -- tls-ca
    -- tls-root-cert
    ```
    ***Các thư mục này được ứng dụng khách Fabric CA sử dụng để:***
    - Lưu trữ các chứng chỉ được cấp khi lệnh đăng ký máy khách Fabric CA đang chạy trên máy chủ TLS CA
    để đăng ký danh tính quản trị viên bootstrap TLS CA. (thư mục tls-ca)
    - Biết nơi cư trú của chứng chỉ gốc TLS CA cho phép ứng dụng khách Fabric CA giao tiếp với TLS
    Máy chủ CA. (thư mục tls-root-cert)

    6.1. Sao chép tệp chứng chỉ gốc của TLS CA tls_ca/ca-cert.pem, được tạo khi máy chủ TLS CA được khởi động, sang ca_client/tls-root-cert /tls-ca-cert.pem. Lưu ý rằng tên tệp được thay đổi thành tls-ca-cert.pem để làm rõ đây là chứng chỉ gốc từ TLS CA. ***Note***: Chứng chỉ gốc TLS CA này sẽ cần có sẵn trên mỗi hệ thống sử dụng fabric-ca-client  để tương tác với server TLS CA

    6.2. Máy khách Fabric CA cũng cần biết tệp nhị phân máy khách Fabric CA nằm ở đâu. Biến môi trường FABRIC_CA_CLIENT_HOME được sử dụng để đặt vị trí: `export FABRIC_CA_CLIENT_HOME=<FULLY-QUALIFIED-PATH-TO-FABRIC-CA-BINARY>`

    Nếu đang ở thư mục chứa file binary fabric-ca-client tức là thư mục ca_client trong hướng dẫn này thì sử dụng câu lệnh: `export FABRIC_CA_CLIENT_HOME=$PWD`

    6.3. Sử dụng CLI ứng dụng khách Fabric CA để đăng ký danh tính quản trị viên bootstrap TLS CA. Chạy lệnh:
    ```
    ./fabric-ca-client enroll -d -u https://<ADMIN>:<ADMIN-PWD>@<CA-URL>:<PORT> --tls.certfiles <RELATIVE-PATH-TO-TLS-CERT> --enrollment.profile tls --mspdir tls-ca/tlsadmin/msp
        Thay thế:
        • <ADMIN> - với quản trị viên TLS CA được chỉ định trên lệnh init.
        • <ADMIN-PWD> - với mật khẩu quản trị TLS CA được chỉ định trong lệnh init.
        • <CA-URL> - với tên máy chủ được chỉ định trong phần csr của tệp .yaml cấu hình TLS CA.
        • <PORT> - với cổng mà TLS CA đang nghe.
        • <RELATIVE-PATH-TO-TLS-CERT> - với đường dẫn và tên của tệp chứng chỉ TLS gốc mà bạn đã sao chép từ TLS CA của mình. Đường dẫn này có liên quan đến FABRIC_CA_CLIENT_HOME. Nếu bạn đang làm theo cấu trúc thư mục trong hướng dẫn này, nó sẽ là tls-root-cert/tls-ca-cert.pem.
    ```
    ***Ví dụ câu lệnh chạy trong hướng dẫn này:***
    ```
    ./fabric-ca-client enroll -d -u https://admintls:admintlspw@sword.com:6666 --tls.certfiles tls-root-cert/tls-ca-cert.pem --enrollment.profile tls --mspdir tls-ca/tlsadmin/msp
    ```


 ### II. Xây dựng một Organization CA

1. Đăng ký định danh TLS cho Organization CA

    - Máy chủ TLS CA được khởi động với danh tính quản trị viên bootstrap (admintls) có đầy đủ đặc quyền quản trị viên cho máy chủ. Một trong những khả năng chính của quản trị viên là khả năng đăng ký danh tính mới. Mỗi nút trong tổ chức (người đặt hàng, đồng nghiệp, CA tổ chức) sẽ giao dịch trên mạng cần phải được đăng ký với TLS CA, để sau đó mỗi nút có thể đăng ký để nhận chứng chỉ TLS của chúng. Do đó, trước khi thiết lập CA tổ chức, chúng ta cần sử dụng TLS CA để đăng ký và đăng ký danh tính bootstrap CA của tổ chức để lấy chứng chỉ TLS và khóa riêng tư của tổ chức đó. Người dùng quản trị bootstrap CA của tổ chức sẽ được đặt tên là admin_ca_org1 trong bước tiếp theo, do đó ta sẽ tạo danh tính TLS cho CA tổ chức bằng cách sử dụng cùng tên. Lệnh sau đăng ký danh tính bootstrap CA của tổ chức admin_ca_org1 với mật khẩu admin_ca_org1_pw với TLS CA.

    - Tạo một folder trong ca_client/tls-ca/ để lưu trữ danh tính TLS cho tổ chức CA muốn đăng ký. Trong hướng dẫn này thì nó là org1-ca.

    ***Register một tổ chức ca bằng câu lệnh:***
    ```
    ./fabric-ca-client register -d --id.name <ADMIN_ORG_CA> --id.secret <ADMIN_ORG_CA_PW> -u https://<ADMIN>:<ADMIN-PWD>@<CA-URL>:<PORT> --tls.certfiles <RELATIVE-PATH-TO-TLS-CERT> --mspdir tls-ca/tlsadmin/msp

        Thay thế:
        • <ADMIN_ORG_CA> - Usename quản trị viên tổ chức CA muốn đăng ký.
        • <ADMIN_ORG_CA_PW> - Mật khẩu quản trị viên tổ chức CA muốn đăng ký.
        • <ADMIN> - Usename quản trị viên TLS CA.
        • <ADMIN-PWD> - Mật khẩu quản trị TLS CA .
        • <CA-URL> - với tên máy chủ được chỉ định trong phần csr của tệp .yaml cấu hình TLS CA.
        • <PORT> - với cổng mà TLS CA đang nghe.
        • <RELATIVE-PATH-TO-TLS-CERT> - với đường dẫn và tên của tệp chứng chỉ TLS gốc mà bạn đã sao chép từ TLS CA của mình. Đường dẫn này có liên quan đến FABRIC_CA_CLIENT_HOME. Nếu bạn đang làm theo cấu trúc thư mục trong hướng dẫn này, nó sẽ là tls-root-cert/tls-ca-cert.pem.
    ```
    ***Ví dụ trong hướng dẫn này câu lệnh đăng ký danh tính TLS cho tổ chức CA có tên org1:***
    ```
    ./fabric-ca-client register -d --id.name admin_ca_org1 --id.secret admin_ca_org1_pw -u https://sword:6666 --tls.certfiles tls-root-cert/tls-ca-cert.pem --mspdir tls-ca/tlsadmin/msp
    ```
    ***Enroll tổ chức CA***
    ```
    ./fabric-ca-client enroll -d -u https://<ADMIN_ORG_CA>: <ADMIN_ORG_CA_PW>@ <CA-URL>:<PORT> --tls.certfiles <RELATIVE-PATH-TO-TLS-CERT> --enrollment.profile tls --csr.hosts '<LIST_URL_IN_ORG_CA' --mspdir <RELATIVE-PATH-TO-SAVE-TLS-CERT>
        Thay thế:
        • <ADMIN_ORG_CA> - Usename quản trị viên tổ chức CA muốn đăng ký.
        • <ADMIN_ORG_CA_PW> - Mật khẩu quản trị viên tổ chức CA muốn đăng ký.
        • <ADMIN> - Usename quản trị viên TLS CA.
        • <ADMIN-PWD> - Mật khẩu quản trị TLS CA .
        • <CA-URL> - với tên máy chủ được chỉ định trong phần csr của tệp .yaml cấu hình TLS CA.
        • <PORT> - với cổng mà TLS CA đang nghe.
        • <LIST_URL_IN_ORG_CA - Danh sách các url sử dụng trong tổ chức CA
        • <RELATIVE-PATH-TO-TLS-CERT> - với đường dẫn và tên của tệp chứng chỉ TLS gốc mà bạn đã sao chép từ TLS CA của mình. Đường dẫn này có liên quan đến FABRIC_CA_CLIENT_HOME. Nếu bạn đang làm theo cấu trúc thư mục trong hướng dẫn này, nó sẽ là tls-root-cert/tls-ca-cert.pem.
        • <RELATIVE-PATH-TO-SAVE-TLS-CERT> - đường dẫn tạo thư mục lưu chứng chỉ định danh được TLS CA cấp cho tổ chức CA đăng ký. Ở trong ví dụ này thì nó là 
    ```
    ***Enroll tổ chức CA vừa register***
        ```
        ./fabric-ca-client enroll -d -u https://admin_ca_org1:admin_ca_org1_pw@sword:6666 --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'sword, localhost' --mspdir tls-ca/org1_ca_admin/msp
        ```
    **Note:** Để tiện cho việc thực hiện ở bước tám thì khóa bí mật được tạo ra trong /tls-ca/org1_ca_admin/msp/keystore/ sẽ được đổi tên thành key.pem

2. Khởi tạo CA

    ***Ở bước này ta sẽ giả định tạo CA cho một tổ chức có tên là Org1***
    - Tạo một folder cùng cấp với folder ca_client để chứa chương trình, ở ví dụ này thì đó là ca_org1
    - Sao chép file binary fabric-ca-server vào folder này.
    - Chạy câu lệnh sau để khởi tạo server cho CA:
        ```
            ./fabric-ca-server init -b <ADMIN_USER>:<ADMIN_PWD>
            Với:
            - <ADMIN_USER>: Thiết lập tài khoản cho admin của tổ chức CA
            - <ADMIN_PWD>: Mật khẩu cho tài khoản này
        ```
        ***Trong ví dụ này ta sẽ cài đặt tài khoản của admin là adminorg1 và mật khẩu là adminorg1pw, câu lệnh sẽ như sau:***
        `./fabric-ca-server init -b adminorg1:adminorg1pw`

    - Sau khi chạy câu lệnh trên thì những file và thư mục cần thiết cho việc khởi tạo server CA đã được tự động sinh ra. Ở đây ta cần chú ý đến file fabric-ca-server-config.yaml và cần chỉnh sửa tối thiểu những thông tin cần thiết cho file này như sau:
        • port - Nhập cổng mà bạn muốn sử dụng cho máy chủ này. Trong hướng dẫn này sử dụng 9998, nhưng bạn có thể chọn port của mình.
    
        • tls.enabled - Bật TLS bằng cách đặt giá trị này thành true.
    
        • tls.certfile và tls.keystore - Nhập đường dẫn tương đối và tên tệp cho chứng chỉ có chữ ký TLS CA và khóa riêng tư được tạo khi quản trị viên bootstrap cho CA này được đăng ký với TLS CA. Chứng chỉ đã ký, cert.pem và khóa cá nhân được tạo ở bước 7 bằng ứng dụng khách Fabric CA và có thể được tìm thấy trong ca_client/tls-ca/org1_ca_admin/msp/signcerts/cert.pem. Khóa cá nhân được đặt trong ca_client/tls-ca/org1_ca_admin/msp/keystore/key.pem 
    
        • ca.name - Đặt tên cho tổ chức CA bằng cách chỉ định một giá trị trong tham số này, ví dụ: ca_org1.
    
        • csr.hosts - Thông thường, thông số này phải là tên máy chủ và địa chỉ ip nơi máy chủ này đang chạy để nó có thể được đưa vào chứng chỉ TLS Tên thay thế chủ đề, tuy nhiên trong trường hợp này máy chủ sẽ không tạo chứng chỉ TLS của chính nó (nó đã được tạo đã từ TLS CA) và do đó không cần cấu hình.
    
        • csr.ca.pathlength: Trường này được sử dụng để giới hạn phân cấp chứng chỉ CA. Đặt giá trị này thành 1 cho CA gốc có nghĩa là CA gốc có thể cấp chứng chỉ CA trung gian, nhưng những CA trung gian này lại không thể cấp chứng chỉ CA khác. Nói cách khác, CA trung gian không thể đăng ký các CA trung gian khác, nhưng nó có thể cấp chứng chỉ đăng ký cho người dùng. Giá trị mặc định là 1.
    
        • Sign.profiles.ca.caconstraint.maxpathlen - Trường này đại diện cho số lượng tối đa chứng chỉ trung gian không tự cấp có thể theo sau chứng chỉ này trong chuỗi chứng chỉ. Nếu đây sẽ là máy chủ mẹ cho CA trung gian và bạn muốn CA trung gian đó hoạt động như CA mẹ cho một CA trung gian khác, CA gốc này cần đặt giá trị này lớn hơn 0 trong tệp .yaml cấu hình. Xem hướng dẫn cho phần ký. Giá trị mặc định là 0.

    ***=> Vậy là ta đã cài đặt xong CA cho một tổ chức, ở bước tiếp theo ta sẽ đăng ký tài khoản admin cho tổ chức CA mới tạo này và thực hiện đăng ký định danh cho một người dùng thuộc tổ chức.***

3. Enroll tài khoản admin và đăng ký danh tính người dùng cho tổ chức CA vừa tạo
    
    - Enroll tài khoản admin của tổ chức CA
    ```
    ./fabric-ca-client enroll -d -u https://<ADMIN>:<ADMIN-PWD>@<CA-URL>:<PORT> --tls.certfiles <RELATIVE-PATH-TO-TLS-CERT> --mspdir <RELATIVE-PATH-TO-SAVE-MSP>

    Thay thế:
        • <ADMIN> - với quản trị viên CA tổ chức được chỉ định trên lệnh init.
        • <ADMIN-PWD> - với mật khẩu quản trị CA của tổ chức được chỉ định trong lệnh init.
        • <CA-URL> - với tên máy chủ cho tổ chức CA.
        • <PORT> - với cổng mà tổ chức CA đang lắng nghe.
        • <RELATIVE-PATH-TO-TLS-CERT> - với đường dẫn đến tệp tls-ca-cert.pem mà bạn đã sao chép từ TLS CA.
        • <RELATIVE-PATH-TO-SAVE-MSP> - đường dẫn đến vị trí bạn định lưu msp cho tài khoản admin.
        Trong trường hợp này, tham số -d chạy máy khách ở chế độ DEBUG, rất hữu ích để gỡ lỗi lệnh.
    ```
    ***Trong hướng dẫn này thì câu lệnh sẽ như sau:***
    ```
    ./fabric-ca-client enroll -d -u https://adminorg1:adminorg1pw@sword:9998 --tls.certfiles tls-root-cert/tls-ca-cert.pem --mspdir org1-ca/admin/msp
    ```

    - Register định danh cho một người dùng trong tổ chức
    ```
    ./fabric-ca-client register -d --id.name <ID_NAME> --id.secret <ID_SECRET> -u <CA_URL> --mspdir <CA_ADMIN> --id.type <ID_TYPE> --id.attrs $ID_ATTRIBUTE --tls.certfiles <TLSCERT>

    Trong đó:
        • ID_NAME: ID đăng ký của danh tính. Tên này sẽ được cấp cho người dùng , người sẽ sử dụng nó khi đăng ký.
        
        • ID_SECRET: Bí mật (tương tự như mật khẩu) cho danh tính. Bí mật này cũng sẽ được cung cấp cho người dùng cùng với ID đăng ký để sử dụng khi đăng ký.
        
        • CA_URL: URL của CA, theo sau là cổng 7054 (trừ khi cổng mặc định đã được thay đổi).
    
        • CA_ADMIN: Đường dẫn đến vị trí của các chứng chỉ dành cho quản trị viên của CA. Được tạo khi ra sau khi enroll tài khoản admin CA ở bước trên.
        
        • ID_TYPE: Loại (hoặc vai trò) của danh tính. Có thể có bốn kiểu: peer, orderer, admin và client (được sử dụng cho các ứng dụng). Type này phải được liên kết với NodeOU có liên quan. Nếu NodeOU không được sử dụng, bạn có thể bỏ qua type và --id.type cờ.
    
        • ID_ATTRIBUTE: Bất kỳ thuộc tính nào được chỉ định cho danh tính này. Các thuộc tính này cũng có thể được thêm vào dưới dạng một mảng JSON, do đó, $ ID_ATTRIBUTE không có nghĩa là đại diện cho một thuộc tính đơn lẻ mà là bất kỳ và tất cả các thuộc tính, nên được đặt trong lệnh đăng ký sau cờ --id.attrs.
        
        • TLSCERT: Đường dẫn gốc đến chứng chỉ đã ký gốc TLS CA của bạn (được tạo khi tạo TLSCA).
    ```
    ***Câu lệnh sử dụng trong hướng dẫn này để đăng ký một định danh cho một admin:***
    ```
        ./fabric-ca-client register -d --id.name org1user --id.secret org1userpw -u https://sword:9998 --mspdir ./org1-ca/admin/msp --id.type admin --tls.certfiles tls-root-cert/tls-ca-cert.pem --csr.hosts 'sword'
    ```

    - Enroll định danh cho một người dùng trong tổ chức
    ```
        ./fabric-ca-client enroll -u https://<ENROLL_ID>:<ENROLL_SECRET><@CA_URL>:<PORT> --mspdir <MSP_FOLDER> --csr.hosts <CSR_HOSTNAME> --tls.certfiles $TLS_CERT

        Tong đó:
        • ENROLL_ID: ID đăng ký được chỉ định khi đăng ký danh tính này. ID này sẽ phải được thông báo cho người dùng danh tính này ngoài băng tần.
        • ENROLL_SECRET: Bí mật đăng ký đã được chỉ định khi đăng ký danh tính này. Điều này sẽ phải được thông báo cho người dùng của danh tính này ngoài băng tần.
        • CA_URL: URL của CA.
        • PORT: Cổng được CA mà bạn đăng ký sử dụng.
        • MSP_FOLDER: Đường dẫn đến MSP (MSP cục bộ, nếu đăng ký một nút hoặc MSP tổ chức, nếu đăng ký quản trị viên) trên hệ thống tệp. Nếu bạn không chỉ định cờ -mspdir để chỉ định một vị trí, các chứng chỉ sẽ được đặt trong một thư mục có tên msp tại vị trí hiện tại của bạn (nếu thư mục này chưa tồn tại, nó sẽ được tạo).
        • CSR_HOSTNAME: Chỉ liên quan đến danh tính nút, điều này sẽ mã hóa tên miền của nút.
        • TLS_CERT: Đường dẫn tương đối đến chứng chỉ gốc TLS CA đã ký của TLS CA được liên kết với tổ chức này.

    ```
    ***Trong hướng dẫn này thì câu lệnh sẽ như sau:***
    ```
    ./fabric-ca-client enroll -u https://org1user:org1userpw@sword:9998 --mspdir ./org1/msp --csr.hosts 'sword' --tls.certfiles tls-root-cert/tls-ca-cert.pem
    ```