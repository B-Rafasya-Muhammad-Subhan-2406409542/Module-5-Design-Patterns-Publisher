# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [X] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [X] Commit: `Create Subscriber model struct.`
    -   [X] Commit: `Create Notification model struct.`
    -   [X] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [X] Commit: `Implement add function in Subscriber repository.`
    -   [X] Commit: `Implement list_all function in Subscriber repository.`
    -   [X] Commit: `Implement delete function in Subscriber repository.`
    -   [X] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
**1. Do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?**
Dalam kasus BambangShop ini, penggunaan sebuah *single Model struct* (`Subscriber`) sudah cukup dan kita tidak diwajibkan menggunakan *interface* atau *trait*. Hal ini karena semua *subscriber* pada sistem kita memiliki struktur dan perilaku yang identik (seragam), yaitu entitas web eksternal yang menerima notifikasi melalui *HTTP POST request* ke `url` masing-masing. *Interface* (atau *trait*) pada *Observer pattern* umumnya dibutuhkan jika *Publisher* harus memberitahu berbagai jenis *Subscriber* yang berbeda-beda cara penanganannya (misal: ada yang via Email, via SMS, atau via Webhook). Karena di sistem kita hanya ada satu metode notifikasi (Webhook via URL), *single struct* sangat memadai dan membuat kode lebih simpel.

**2. Is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?**
Penggunaan `DashMap` (map/dictionary) sangat disarankan dan jauh lebih baik daripada `Vec` (list) untuk kasus ini. Karena `id` atau `url` sifatnya unik, kita akan sering melakukan operasi pencarian (*lookup*) dan penghapusan (*delete*) berdasarkan `url` tersebut (misalnya saat fitur *unsubscribe* dipanggil). Jika menggunakan `Vec`, kita harus melakukan iterasi satu per satu secara linear (*time complexity* $O(n)$) untuk menemukan *subscriber* yang tepat. Sebaliknya, `DashMap` memungkinkan pencarian, penambahan, dan penghapusan data berdasarkan *key* (`url`) secara instan dengan *time complexity* rata-rata $O(1)$.

**3. Do we still need DashMap or we can implement Singleton pattern instead?**
Kita **tetap membutuhkan DashMap**. *Singleton pattern* dan `DashMap` memecahkan dua masalah yang berbeda. *Singleton pattern* (yang di Rust kita capai menggunakan *macro* `lazy_static!`) hanya memastikan bahwa variabel `SUBSCRIBERS` diinisialisasi satu kali dan dapat diakses secara global. Namun, *Singleton* tidak menjamin keamanan konkurensi (*thread-safety*). Karena framework Rocket bersifat *multi-threaded* (menangani banyak *request* HTTP secara bersamaan), banyak *thread* bisa saja mencoba membaca dan mengubah data *subscriber* secara bersamaan. `DashMap` digunakan secara spesifik karena ia adalah implementasi *HashMap* yang dirancang aman untuk skenario konkurensi tersebut (*thread-safe*), menghindari *race conditions* atau data *corrupted* yang akan ditolak oleh *compiler* ketat Rust.

#### Reflection Publisher-2
**1. Why do we need to separate "Service" and "Repository" from a Model?**
Pemisahan ini dilakukan untuk menerapkan prinsip *Single Responsibility Principle* (SRP) dan *Separation of Concerns* dalam arsitektur perangkat lunak. 
- **Model** murni digunakan sebagai *data container* atau representasi struktur data.
- **Repository** (Data Access Layer) bertanggung jawab khusus untuk berinteraksi dengan media penyimpanan (*database*, memori, atau API eksternal), memisahkan query data dari logika aplikasi.
- **Service** (Business Logic Layer) bertanggung jawab atas aturan bisnis dan memproses data, menjembatani *Controller* dan *Repository*.
Dengan memisahkan ketiganya, kode menjadi lebih terstruktur, modular, mudah di-*maintain*, dan jauh lebih mudah untuk diuji (*unit testing*) karena setiap komponen memiliki satu tugas spesifik.

**2. What happens if we only use the Model? How do interactions between models affect code complexity?**
Jika kita hanya menggunakan *Model* (sering disebut *Fat Model*), seluruh logika bisnis dan operasi *database* akan menumpuk di dalam satu *struct*. Hal ini akan menciptakan *tight coupling* (ketergantungan yang tinggi) antar komponen. Misalnya, *Model* `Product` harus tahu cara memanggil *Model* `Notification`, dan `Notification` harus tahu cara mencari data di *database* `Subscriber`. Akibatnya, kompleksitas kode akan meledak (*spaghetti code*). Jika ada perubahan pada cara penyimpanan data (misal dari memori ke PostgreSQL), kita harus merombak seluruh *Model* yang memuat fungsi *database* dan logika bisnis secara bersamaan, sehingga risiko munculnya *bug* sangat besar.

**3. Tell us how Postman helps you to test your current work and list useful features.**
Postman sangat membantu dalam menguji REST API yang saya buat tanpa harus membangun antarmuka (*frontend*) terlebih dahulu. Saya bisa menyimulasikan berbagai *HTTP request* (GET, POST, dll.) dan melihat *response*-nya secara langsung, baik *body* datanya (dalam bentuk JSON) maupun *HTTP status code* (seperti 201 Created atau 404 Not Found).
Beberapa fitur Postman yang menurut saya sangat berguna untuk tugas maupun proyek rekayasa perangkat lunak ke depannya adalah:
- **Collections:** Untuk mengelompokkan *endpoint-endpoint* yang saling berkaitan agar lebih terorganisir.
- **Environments & Variables:** Memungkinkan kita berpindah dari environment *development* (localhost) ke *production* dengan cepat tanpa harus mengubah URL secara manual di setiap *request*.
- **Tests & Pre-request Scripts:** Sangat berguna untuk melakukan *automated testing* sederhana pada API yang sudah dibuat.

#### Reflection Publisher-3
