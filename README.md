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