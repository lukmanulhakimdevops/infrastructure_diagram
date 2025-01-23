Infrastructure Diagram Flow

Proses Alur Sistem (Flow) 
1. User Interaction Layer
Proses Flow:   
o Pengguna mengirimkan request dari Mobile/ Desktop/ API ke Content Delivery Network (CDN). Hal ini bisa dilakukan melalui HTTPS, dan CDN bertugas untuk menyediakan cache dan konten statis (seperti React JS Static Files) yang lebih cepat diakses oleh pengguna. 
o HA di layer ini tercapai dengan menggunakan Multiple CDN Edge Locations yang mendistribusikan request ke server terdekat dari pengguna, mengurangi latensi dan meningkatkan ketersediaan.

2. Application Load Balancer (ALB) Layer 
Proses Flow: 
o Setelah request mencapai CDN, request diteruskan ke Application Load Balancer (ALB) yang berfungsi untuk Layer 7 Routing, yaitu routing berdasarkan HTTP(S) request (misalnya, URL atau header). 
o SSL Termination: ALB mengelola enkripsi dan dekripsi SSL untuk memastikan koneksi aman. 
o WAF (Web Application Firewall): Memfilter request yang mencurigakan dan menambah lapisan keamanan dengan melakukan inspeksi terhadap traffic yang masuk. 
o Ingress Controller (I): Di dalam Kubernetes, Ingress Controller bertanggung jawab untuk menerima dan memproses traffic yang menuju ke aplikasi di dalam cluster.

High Availability (HA): 
o ALB dan Ingress Controller dapat dikelola untuk HA dengan load balancing otomatis. Jika satu instance ALB atau Ingress Controller down, yang lain akan menangani request. 
o Kubernetes juga mendukung HA di level Ingress Controller dan ALB, dengan konfigurasi yang membuat service tetap tersedia meskipun ada kegagalan di sebagian node atau service. 

3. Kubernetes Cluster (Application Services) 
Proses Flow: 
o Setelah traffic diteruskan oleh Ingress Controller, Frontend (React JS Static Files) berkomunikasi dengan Backend Services (Gateway, Publish, Consume) di dalam cluster. Komunikasi antar layanan ini bisa melibatkan API calls atau messaging untuk berbagai fungsi aplikasi.

Scaling: 
o Horizontal Pod Autoscaler (HPA): Pada level aplikasi, HPA secara otomatis scales jumlah pod berdasarkan CPU usage atau custom metrics lainnya. Misalnya, jika Backend Services atau Frontend menerima banyak trafik, HPA akan menambah jumlah pod untuk menangani beban lebih.

o Vertical Pod Autoscaler (VPA): Mengatur resource limits pada pod, seperti CPU dan memori. Jika load meningkat, VPA dapat menambah resource pada pod yang sudah ada, daripada menambah pod baru. Hal ini membantu dalam mengelola pod yang memerlukan lebih banyak kapasitas untuk memproses lebih banyak data.

High Availability (HA): 
o Layanan backend dan frontend dideploy dengan replica sets untuk 
memastikan HA. Jika salah satu pod mati, pod lain akan langsung mengambil 
alih tanpa mengganggu ketersediaan layanan. 
o StatefulSets atau Persistent Volumes digunakan untuk database yang 
membutuhkan state seperti MySQL, Redis, atau Pub/Sub untuk memastikan 
data tetap tersedia bahkan saat pod berganti. 

4. Database Services 
Proses Flow:
o MySQL: Backend service berkomunikasi dengan MySQL untuk mendapatkan 
atau menyimpan data. 
o Redis: Backend juga menggunakan Redis untuk caching atau penyimpanan 
data sementara yang cepat diakses. 
o Pub/Sub: Untuk komunikasi antar komponen atau layanan lain, digunakan 
Pub/Sub service (misalnya, Kafka atau Google Pub/Sub).

Scaling: 
o Scaling MySQL: MySQL di Kubernetes dapat di-scaling dengan menggunakan 
StatefulSets dan Persistent Volumes untuk mengelola storage secara 
dinamis. Anda bisa menggunakan read replicas untuk meningkatkan read 
scalability. 
o Redis Scaling: Redis juga dapat di-scaling secara horizontal, dengan 
menambahkan lebih banyak Redis Pods untuk distribusi load, serta 
menggunakan Redis Cluster untuk high availability dan scaling lebih lanjut. 
o Pub/Sub Scaling: Pub/Sub service, seperti Kafka, dapat disesuaikan secara 
horizontal dengan menambah jumlah partition atau broker untuk menangani 
jumlah pesan yang lebih tinggi. 

5. Monitoring Ecosystem
Proses Flow: 
o Infrastructure Monitoring: Memantau kondisi keseluruhan infrastruktur 
(seperti nodes, pods, jaringan, dan storage) menggunakan alat seperti 
Prometheus atau Datadog. 
o Log Monitoring: Menyimpan dan menganalisis log aplikasi dan container 
menggunakan alat seperti ELK stack (Elasticsearch, Logstash, Kibana) atau 
Grafana Loki. 
o Application Monitoring: Memantau metrik aplikasi (seperti latency API, 
jumlah request, error rate) untuk memastikan performa aplikasi tetap optimal. 
Alat yang umum digunakan di Kubernetes adalah Prometheus dengan 
Grafana sebagai dashboard-nya. 
o Alerting & Dashboard: Pengguna dapat membuat alert berdasarkan metrik 
yang dikumpulkan. Jika suatu metrik melampaui threshold (misalnya, CPU 
usage yang tinggi), sistem akan mengirimkan notifikasi ke admin atau tim 
DevOps.

Housekeeping: 
o Log Rotation dan Retention Policies: Untuk menjaga kebersihan dan efisiensi penyimpanan log, log dapat diputar secara berkala dan disimpan hanya dalam jangka waktu tertentu. Log yang lebih lama dapat diarsipkan atau dihapus.

o Pod Garbage Collection: Kubernetes juga melakukan housekeeping pada pod-pod yang sudah tidak aktif atau selesai diproses. 
o Node Autoscaling: Kubernetes dapat menambah atau mengurangi jumlah node berdasarkan beban cluster. Cluster Autoscaler memantau penggunaan sumber daya dan memutuskan untuk menambah atau mengurangi node.

6. Interaksi dan Tugas-Tugas Lainnya
Interaksi Antar Layanan: 
o Frontend dan Backend saling berinteraksi untuk mendapatkan data atau melakukan proses bisnis. Misalnya, frontend memanggil backend API untuk mengambil data dari MySQL atau Redis. 
o Backend Services berkomunikasi dengan database untuk CRUD data dan berinteraksi dengan Pub/Sub untuk mendistribusikan pesan antar layanan. 

Monitoring: Metrik aplikasi dan log aplikasi akan mengalir ke sistem monitoring (seperti Prometheus dan Grafana), yang kemudian menampilkan dashboards dengan berbagai metrik, dari performa aplikasi hingga status database dan infrastruktur. 

Alerting: Jika terdeteksi masalah, sistem akan mengirimkan alert, yang bisa berupa email, Slack notification, atau notifikasi lainnya ke tim yang bertanggung jawab, untuk segera menangani masalah tersebut.
