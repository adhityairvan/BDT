---


---

<h1 id="implementasi-redis-cluster">Implementasi Redis Cluster</h1>
<p>Implementasi dan tutorial pembuatan cluster redis.<br>
dokumentasi ini akan dibuat dengan <strong>Elementary OS</strong>, <strong>Vagrant</strong>, <strong>Ubuntu 18.04</strong> sebagai vagrant image nya.</p>
<h2 id="persiapan">Persiapan</h2>
<p>Dalam tahap persiapan, akan dibuat <strong>VagrantFile</strong> yang akan memudahkan dalam pembuatan virtual machine ubuntu.<br>
Akan dibuat 3 Virtual machine</p>

<table>
<thead>
<tr>
<th>clusterName</th>
<th>IP</th>
<th>Role</th>
</tr>
</thead>
<tbody>
<tr>
<td>clusterdb1</td>
<td>192.168.33.11</td>
<td>Master + sentinel</td>
</tr>
<tr>
<td>clusterdb2</td>
<td>192.168.33.12</td>
<td>Slave + sentinel</td>
</tr>
<tr>
<td>clusterdb3</td>
<td>192.168.33.13</td>
<td>Slave + sentinel</td>
</tr>
</tbody>
</table><h3 id="konfigurasi-vagrant">Konfigurasi Vagrant</h3>
<pre><code>Vagrant.configure("2") do |config|
  (1..3).each do |i|
    config.vm.define "rediscluster#{i}" do |node|
      node.vm.hostname = "rediscluster#{i}"
      node.vm.box = "bento/ubuntu-18.04"
      node.vm.network "private_network", ip: "192.168.33.1#{i}"
      node.vm.provider "virtualbox" do |vb|
        vb.name = "rediscluster#{i}"
        vb.gui = false
        vb.memory = "1024"
      end

      node.vm.provision "shell", path: "bootstrap.sh", privileged: false
    end
  end
end
</code></pre>
<h2 id="instalisasi">Instalisasi</h2>
<p>Redis-server akan diinstall disetiap vagrant machine<br>
command yang dijalankan:</p>
<pre><code>sudo apt-get update -y
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:chris-lea/redis-server -y
sudo apt-get install redis-server -y
sudo systemctl enable redis-server.service
</code></pre>
<p>buka firewall untuk port default redis</p>
<pre><code>sudo ufw allow 6379
sudo ufw allow 26379
</code></pre>
<p>untuk mempermudah setup pada setiap virtual machine, membuat provision script yang dijalankan setiap setup virtual machine sangat disarankan</p>
<h2 id="konfigurasi-redis">Konfigurasi Redis</h2>
<p>Ada 2 hal yang perlu kita konfigurasi</p>
<ol>
<li>Redis server</li>
<li>Sentinel Server</li>
</ol>
<p>Redis server akan kita setup menjadi dua tipe, master dan slave.<br>
sedangkan sentinel server akan kita buat di seluruh node yang ada</p>
<h3 id="konfigurasi-master">Konfigurasi Master</h3>
<p>lakukan perintah dibawah untuk mengkonfigurasi master node<br>
edit file konfigurasi yang ada pada directory redis<br>
<strong>redis.conf</strong></p>
<pre><code>#cari dan rubah opsi pada file redis.conf
#comment line yang berisi "bind 127.0.0.1 ::1"
protected-mode no 
</code></pre>
<h3 id="konfigurasi-slave">Konfigurasi Slave</h3>
<p><em><strong>redis.conf</strong></em></p>
<pre><code>protected-mode no
#comment line yang berisi "bind 127.0.0.1 ::1"
slaveof 192.168.33.11 6379
#ganti ip nya agar sesuai dengan ip master node
</code></pre>
<h3 id="konfigurasi-sentinel">konfigurasi Sentinel</h3>
<p>download file default konfigurasi sentinel</p>
<pre><code>wget http://download.redis.io/redis-stable/sentinel.conf
cp sentinel.conf /etc/redis/
</code></pre>
<p>rubah opsi dalam file tsb</p>
<p><em><strong>sentinel.conf</strong></em></p>
<pre><code>sentinel monitor mymaster 192.168.33.11 6379 2
sentinel down-after-milliseconds 5000
sentinel failover-timeout mymaster 10000
</code></pre>
<p>opsi diatas menentukan master node nya dan syarat waktu untuk mengenali sebuah master node mati atau timeout</p>
<p>Buat lah sentinel.conf di setiap node, buat 2 slave node dan satu master node</p>
<h2 id="menjalankan-server-redis">Menjalankan server redis</h2>
<p>jalankan command di bawah pada setiap node</p>
<pre><code>sudo -u redis redis-server /etc/redis/redis.conf &amp;
sudo -u redis redis-server /etc/redis/sentinel.conf &amp;
</code></pre>
<p>`command di atas akan menjalankan sentinel dan server redis<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2006-13-24.png?raw=true" alt="enter image description here"></p>
<h2 id="ujicoba-crud-pada-master">Ujicoba CRUD pada master</h2>
<p>uji coba ini akan dilakukan pada node master dengan memasukan dan melihat key yang ada<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2006-29-54.png?raw=true" alt="enter image description here"></p>
<h2 id="hasil-simulasi-fail-over">Hasil simulasi Fail Over</h2>
<p>menghentikan master node<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2006-35-31.png?raw=true" alt="enter image description here"></p>
<p>respon dari sentinel saat master node mati<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2007-37-38.png?raw=true" alt="enter image description here"></p>
<p>respon pada slave node<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2007-37-07.png?raw=true" alt="respon pada slave node"></p>
<p>salah satu slave node berubah menjadi master node<br>
<img src="https://github.com/adhityairvan/BDT/blob/master/Screenshot%20from%202019-05-15%2007-38-11.png?raw=true" alt="enter image description here"></p>

