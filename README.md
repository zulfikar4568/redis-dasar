# Instalasi

Install dengan docker
```bash
docker-compose up --build -d
```

Masuk ke dalam container
```
docker exec -it redis /bin/sh
```

Ping Redis
```
redis-cli -h localhost
ping
```

# Database

Kita bisa lihat konfigurasi database di redis.conf. Pada contoh ini kita set database sebanyak 16. Yang artinya akan terdapat index 0 - 15 
```bash
...

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16
```

Select Database
```bash
# Gunakan Redis Client
redis-cli -h localhost

select 1
```

# Strings

Redis sebenernya bisa banyak menggunakan berbagai jenis struktur data List, Set, String dll. Yang paling banyak di gunakan adalah string.

```bash
# Set nilai string di key
set key-saya 'Nilai Key Saya'
set key-saya 'Nilai Key Saya2'

# Mengambil value dari key return value atau nil
get key-saya
get tidak-ada

# Check key value ada atau tidak, return 1 (ada) atau 0(tidak ada)
exists key-saya
exists key-saya

# Delete key, return 1 (ada yang ter delete) atau 0 (tidak ada yang ter delete)
del key-saya key-saya2
del tidak-ada

# Menambahkan nilai di belakang dengan Append
append key-saya '100'

# Cari semua key yang cocok dengan pattern yang diberikan
set key-saya 'Nilai Key Saya'
set key-saya2 'Nilai Key Saya2'
keys key-saya* # Contoh cari nilai dengan key awalan "key-saya" 
keys *2 # Contoh ambil key yang akhiran 2
keys * # Print semua key

# Mengambil nilai dengan range tertentu
getrange key-saya 0 4

# Hanya mengubah nilai di renage tertentu
setrange key-saya 0 'value'


# Mengambil multi nilai dari multi key
mget key-saya key-saya2

# Men set multi nilai ke dalam multi key
mset key-saya "nilai key saya ubah" key-saya2 "nilai key saya ubah 2"
```

# Expiration

- Secara default saat kita menyimpan di redis, redis akan menyimpan nya secara permanen, sampai kita menghapus nya.
- Jika kita ingin menghapus otomatis di waktu tertentu kita bisa gunakan expiration, misal kita menyimpan dalam waktu 1 menit, setelah 1 menit akan query kembali ke dalam database

```bash
# Set key expire dalam detik
expire key-saya 10

# Set data + expire waktunya
setex key-saya 10 "Nilai Key Saya"

# Mengetahui life time
ttl key-saya
```

# Increment & Decrement

- Contoh implementasi misal di stok barang pada flash sale
- Operasi Increment & Decrement sekilas sangat mudah dilakukan, hanya tinggal mengupdate data yang di redis dengan data baru (data lama ditambah 1).
- Namun jika operasi dilakukan secara paralel dan dalam waktu yang sangat cepat, hal ini bisa memungkinkan race condition
- Untungnya redis memiliki operasi untuk melakukan increment dan decrement

```bash
# Increment 1 nilai integer di key
incr key-int

# Decrement 1 nilai integer di key
decr key-int

# Increment nilai berdasarkan input integer di key
incrby key-int 5

# Decrement nilai berdasarkan input integer di key
decrby key-int 5
```

# Flush

- Mengosongkan semua data di redis bisa menggunakan flush, sebenarnya bisa saja menggubakan delete namun kurang etis. Atau dengan men shutdown redis nya, tetapi redis akan mengembalikan data nya krena memiliki backup
  
```bash
# Menghapus semua key value di current database
flushdb

# Menghapus key value di seluruh database
flushll
```

# Pipeline

- Perintah yang dikirim dari client ke server redis menggunakan Request/Response protocol
- Artinya tiap request yang dikirim ke server redis, maka redis akan membalasnya secara langsung
- Kadang ada kebutuhan kita mengirim data ke redis dalam jumlah besar, misal ketika ada kasus memindahkan data dari database mysql ke redis
- Jika kita mengirim satu per satu datanya, maka akan butuh waktu lama untuk selesai
- Redis mendukung operasi bulk via pipeline, dimana kita bisa mengirim beberapa perintah sekaligus dalam satu request
- Namun perlu diketahui, server redis tidak akan membalas tiap perintah yang dikirim via pipeline

Pipeline menggunakan redis-cli
```bash
redis-cli --pipe
```


```bash
cd /usr/local/etc/command
cat sets.txt  | redis-cli -h localhost --pipe

#response => 
# All data transferred. Waiting for the last reply...
# Last reply received from server.
# errors: 0, replies: 1000
```

# Transaction

- Seperti pada database relational, redis juga mendukung transaction
- Proses transaction adalah proses dimana kita mengirimkan beberapa perintah, dan perintah tersebut akan dianggap sukses jika semua perintah sukses, jika gagal maka semua perintah harus dibatalkan

```bash
# Tandai perintah yang akn di eksekusi (batch), maka perintah yang akan dimasukan setelahnya menjadi QUEUED
multi
set key-saya 'zul'
set key-saya2 'zulfikar'

# Eksekusi perintah yang berada dalam QUEUED
exec

# Jika salah perintah maka tidak akan bisa di exec
set key-saya
exec # (error) EXECABORT Transaction discarded because of previous errors.

# Membatalkan perintah yang ada di QUEUED
discard
```

# Monitor

- Kadang ada kasus kita ingin mendebug aplikasi saat berkomunikasi dengan redis
- Redis memiliki fitur monitor, yaitu fitur untuk memonitor semua request yang masuk ke redis server
- Dengan fitur ini kita bisa mudah mendebug jika ternyata ada perintah yang salah yang dikirim oleh aplikasi kita ke redis server

Buka 2 terminal redis-cli
```bash
# Terminal 1
monitor

# Terminal 2 Masukan perintah apapun
set zul 10000
```


# Server Information

- Kadang kita butuh mendapatkan informasi dan statistik redis server
- Seperti jumlah memory yang sudah terpakai, konfigurasi dan lain-lain
- Redis memiliki fitur ini, sehingga kita sangat mudah untuk mendapat informasi server dan memonitor nya


```bash
# Mengambil informasi dan statistik server
info
info keyspace
info memory

# Mengambil data konfigurasi
config get *
config get databases

# Mengembalikan perintah perintah yang lambat di eksekusi oleh redis server
slowlog get
slowlog get 5 # Mengambil 5 query yang lambat
```

# Client Connection

- Redis menyimpan semua informasi client di server
- Hal ini memudahkan kita untuk melihat daftar client, dan juga mengecek jika ada anomali, seperti terlalu banyak koneksi client ke redis

```bash
# Mengambil list client yang aktif
client list

# Mengambil id yang saat ini digunakan
client id

# Kill connection of client
client kill 127.0.0.1:35140 # client kill ip:port

# Jika kita kill diri kita sendiri maka akan auto reconnet dengan id yang berbeda
```

# Security

- Secara default, ketika kita menyalakan redis server, redis server akan mendengarkan request dari semua network interface. Ini sangat berbahaya, karena bisa jadi redis terekspos secara public
- Namun, redis punya second layer untuk pengecekan koneksi, yaitu mode protected, secara default mode protectednya aktif, artinya walaupun redis bisa diakses dari manapun, tapi redis hanya mau menerima request dari 127.0.0.1 (localhost)

Contoh configurasi Network
```bash
################################## NETWORK #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all available network interfaces on the host machine.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only on the
# IPv4 loopback interface address (this means Redis will only be able to
# accept client connections from the same host that it is running on).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT OUT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1
```

Contoh konfigurasi Protected Mode
```bash
# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes
```

Mencoba connect redis server dari container yang berbeda (redis client)
```bash
# Terminal 1
docker exec -it redis /bin/sh
```

```bash
#Terminal 2
docker exec -it redis-client /bin/sh
redis-cli -h redis

# Jika kita aktifkan bind bind 127.0.0.1 maka akan "Could not connect to Redis at redis:6379: Connection refused not connected> "
```

Jadikan comment `bind 127.0.0.1 `, lalu restart kedua container
```bash
# Terminal 1
docker exec -it redis /bin/sh
```

```bash
#Terminal 2
docker exec -it redis-client /bin/sh
redis-cli -h redis

# Maka akn terhubung
# redis:6379>
```

Pada redis container kita implementasi `protected-mode yes` artinya semua interface akan di terima, namun akan aktif ketika kita tidak implementasi bind network

```bash
 # Terminal 2
 ping 
 # (error) DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
```

# Authentication

- Authentication adalah proses verifikasi identitas untuk memastikan bahwa yang mengakses adalah identitas yang benar
- Redis memiliki fitur authentication, dan kita bisa menambahkannya di file konfigurasi di server redis
- Namun perlu diingat, proses authentication di redis itu sangat cepat, jadi pastikan gunakan password sepanjang mungkin agar tidak mudah untuk di brute force 

Configurasi awalnya seperti di bawah, yang artinya redis akan mendengarkan request dari semua network interface, tetapi send layer nya aktif (protected mode)
```bash
# bind 127.0.0.1
protected-mode yes
```

Contoh configurasi Auth di redis
```bash
...# Redis ACL users are defined in the following format:
#
#   user <username> ... acl rules ...
#
# For example:
#
#   user worker +@list +@connection ~jobs:* on >ffa9203c493aa99
#
# The special username "default" is used for new connections. If this user
# has the "nopass" rule, then new connections will be immediately authenticated
# as the "default" user without the need of any password provided via the
# AUTH command. Otherwise if the "default" user is not flagged with "nopass" ...
```

Tambahkan di config redis
```bash
user zulfikar on +@all >zulfikarpassword
```
```bash
# Terminal 1
docker exec -it redis /bin/sh
redis-cli -h localhost
```

```bash
#Terminal 2
docker exec -it redis-client /bin/sh
redis-cli -h redis
auth zulfikar zulfikarpassword
# Maka akan tetap denied
```

Tambahkan config di redis
```bash
user default on +@connection
user zulfikar on +@all >zulfikarpassword
```
Lalu ping kembali
```bash
docker exec -it redis-client /bin/sh
redis-cli -h redis
ping
#response => (error) NOAUTH Authentication required.

auth zulfikar zulfikarpassword
ping
# Maka akan terhubung, respose => PONG
```

# Authorization
- Authorization adalah prose memberi hak akses terhadap identitas yang telah berhasil melewati proses authentication
- Redis mendukung hal ini, jadi kita bisa membatasi hak akses apa saja yang bisa dilakukan oleh identitas yang kita buat
- https://redis.io/topics/acl
- https://redis.io/commands/acl-cat

```
+ = boleh
- = tidak boleh
@<category>
~ = keys
```

Melihat category
```bash
acl cat
acl cat read 
```

Tambahkan di config redis
```bash
user default on +@connection
user isnaen on +@read ~* >isnaenpassword
user irpan on +@read +@set ~* >irpanpassword
user jack on +@all -@set ~* >jackpassword # Jack bisa semua kecuali set
user arif on +@read ~key-arif* >arifpassword
user zulfikar on +@all ~* >zulfikarpassword
```
Restart Container 
```bash
docker exec -it redis-client /bin/sh
redis-cli -h redis
auth zulfikar zulfikarpassword

get key # zulfikar punya akses read
set key-zul val-zul # zulfikar punya akses write
get key-zul # zulfikar punya akses read

auth isnaen isnaenpassword
get key # isnaen punya akses read
set key-zul val-zul # isnaen tidak punya akses write

auth isnaen arifpassword
get key #arif tidak punya akses read key dengan nama key
set key-arif val-arif # arif tidak punya akses write
get key-a #arif tidak punya akses read key dengan nama key-a
get key-arif1 # arif punya akses read key dengan key-arif1
get key-arif2 # arif punya akses read key dengan key-arif2
```

# Persistance
- Media penyimpanan utama redis adalah di memory
- Namun kita bisa menyimpan data di memory redis tersebut di disk jika kita mau
- Namun perlu diingat proses penyimpanan data ke disk redis tidak realtime, dia dilakukan secara scheduler dengan konfigurasi tertentu
- Jadi jangan jadikan redis sebagai media penyimpanan persistence, gunakan redis sebagai database untuk membantu database persistence lainnya

Contoh configuration persistance
```bash
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behavior will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
```

Bisa menggunakan konfigurasi file atau bisa manual menggunakan command cli

```bash
# Synchronous save data to disk
save
# Asynchronous save data to disk
bgsave
```

Data akan tetap utuh jika redis dimatikan

# Ketika Memory Redis Penuh
- Ketika memory redis penuh, maka redis secara default akan mereject semua request penyimpanan data
- Hal ini mungkin menjadi masalah ketika kita hanya menggunakan redis sebagai cache untuk media penyimpanan sementara
- Kadang akan sangat berguna jika memory penuh, redis bisa secara otomatis menghapus data yang sudah jarang digunakan


# Eviction

- Redis mendukung fitur eviction (menghapus data lama, dan menerima data baru)
- Namun untuk mengaktifkan fitur ini, kita perlu memberi tahu redis, maximum memory yang boleh digunakan, dan bagaimana strategi untuk melakukan eviction nya
- https://redis.io/topics/lru-cache

Konfigurasi Max Memory
```bash 
# In short... if you have replicas attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for replica
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
```

Konfigurasi Memory Policy
```bash
# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select one from the following behaviors:
#
# volatile-lru -> Evict using approximated LRU, only keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
```

Contoh Konfigurasi
```bash
maxmemory 100mb
maxmemory-policy allkeys-lfu
```