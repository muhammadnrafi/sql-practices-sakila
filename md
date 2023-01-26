USE sakila;

-- tampilkan total omset bulanan pada tahun 2005, diurutkan dari yang paling kecil
SELECT * FROM payment;
SELECT MONTHNAME(payment_date) AS Bulan, SUM(amount) AS omset FROM payment WHERE YEAR(payment_date) = 2005 GROUP BY Bulan ORDER BY omset;

-- Berapakah jumlah hari peminjaman paling lama di toko rental Sakila?
SELECT MAX(DATEDIFF(DATE(return_date), DATE(rental_date))) AS lama_pinjam_max FROM rental; 

-- Tampilkan aktor yang memiliki nama depan ‘Scarlett’!
SELECT first_name AS Nama_Depan, last_name AS Nama_Belakang FROM actor
WHERE first_name = 'SCARLETT';

-- Tampilkan aktor yang memiliki nama belakang ‘Johansson’!
SELECT first_name AS Nama_Depan, last_name AS Nama_Belakang FROM actor
WHERE last_name = 'JOHANSSON';

-- Berapa banyak nama terakhir aktor (tanpa ada pengulangan/_distint_)?
SELECT COUNT(DISTINCT(last_name)) AS Jumlah_Nama_Belakang FROM actor;

-- Tampilkan 5 nama belakang aktor yang keluar hanya satu kali di database Sakila!¶
with belakang_1 AS (SELECT DISTINCT(last_name) AS Nama_Belakang, COUNT(last_name) AS Jumlah FROM actor
GROUP BY last_name
ORDER BY Jumlah ASC)
SELECT Nama_Belakang FROM belakang_1 LIMIT 5;

-- Tampilkan 5 nama belakang aktor yang keluar lebih dari satu kali di database Sakila!
with belakang_1 AS (SELECT DISTINCT(last_name) AS Nama_Belakang, COUNT(last_name) AS Jumlah FROM actor
GROUP BY last_name
ORDER BY Jumlah DESC)
SELECT Nama_Belakang FROM belakang_1 LIMIT 5;

-- Berapa rata-rata durasi film di database Sakila?
SELECT AVG(length) FROM film;

-- Tampilkan rata-rata durasi film di tiap kategori film!
SELECT category.name AS Category, AVG(film.length) AS Rata_Durasi
    FROM category
    JOIN film_category
    ON category.category_id = film_category.category_id
    JOIN film
    ON film_category.film_id = film.film_id
    GROUP BY category.name
    ORDER BY Rata_Durasi DESC;
    
-- Tampilkan kategori film yang memiliki rata-rata durasi (_average of length_) di atas rata-rata kategori film lainnya!
WITH categ_durasi AS (SELECT category.name AS Category, AVG(film.length) AS Rata_Durasi
    FROM category
    JOIN film_category
    ON category.category_id = film_category.category_id
    JOIN film
    ON film_category.film_id = film.film_id
    GROUP BY category.name),
    rata AS (SELECT AVG(Rata_Durasi) AS rata_dur FROM categ_durasi)
    
SELECT Category, Rata_Durasi FROM categ_durasi
WHERE Rata_Durasi > (SELECT rata_dur FROM rata)
ORDER BY Rata_Durasi DESC;

-- Berapa banyak customer yang dilayani oleh 2 staff di Sakila Database?
SELECT CONCAT(staff.first_name, staff.last_name) AS nama, COUNT(rental.customer_id) FROM staff, rental
WHERE rental.staff_id = staff.staff_id
GROUP BY staff.first_name;

-- Tampilkan _total order_ (jumlah penyewaan) setiap kategori film!
SELECT category.name AS kategori, COUNT(inventory.inventory_id) total_rental
FROM category
JOIN film_category ON category.category_id = film_category.category_id
JOIN film ON film.film_id = film_category.film_id
JOIN inventory ON inventory.film_id = film.film_id
JOIN rental ON rental.inventory_id = inventory.inventory_id
GROUP BY kategori
ORDER BY total_rental DESC;

-- Tampilkan 5 kota dengan total order (jumlah penyewaan) terbesar!
SELECT city.city AS nama_kota, COUNT(rental.rental_id) AS total_order
FROM customer
JOIN address ON address.address_id = customer.address_id
JOIN city ON city.city_id = address.city_id
JOIN rental ON rental.customer_id = customer.customer_id
GROUP BY nama_kota
ORDER BY total_order DESC
LIMIT 5;

-- Tahun berapa film ‘Academy Dinosaur’ di-_release_?
SELECT title AS Judul, release_year AS Tahun_Release FROM film
WHERE title = 'ACADEMY DINOSAUR';

-- 10 film komedi dengan durasi tersingkat
SELECT film.title, category.name AS category, film.length
    FROM film
    JOIN film_category
    ON film.film_id = film_category.film_id
    JOIN category
    ON film_category.category_id = category.category_id 
    WHERE category.name='comedy'
    ORDER BY film.length
    LIMIT 10;
    
-- kategori film beserta jumlah film tiap kategori & rata-rata harga sewa DVD film tiap kategori
SELECT category.name, count(film_category.film_id) AS jumlah, AVG(film.rental_rate)
    FROM category
    JOIN film_category
    ON category.category_id = film_category.category_id
    JOIN film
    ON film_category.film_id = film.film_id
    GROUP BY category.name
    ORDER BY jumlah DESC;

-- 10 aktor/aktris yang paling banyak membintangi film
SELECT film_actor.actor_id, actor.first_name, actor.last_name, count(distinct(film_actor.film_id)) AS jumlah_movie
    FROM film_actor
    JOIN actor
    ON film_actor.actor_id = actor.actor_id
    GROUP BY film_actor.actor_id
    ORDER BY jumlah_movie DESC, actor.actor_id DESC
    LIMIT 10;

-- kategori film beserta jumlah film yang pernah dibintangi oleh Gina Degeneres
SELECT category.name, count(category.category_id) AS jumlah_movie
    FROM category
    JOIN film_category
    ON category.category_id = film_category.category_id
    JOIN film_actor
    ON film_category.film_id = film_actor.film_id
    WHERE film_actor.actor_id = 107
    GROUP BY category.name;

-- judul film sci-fi yang pernah dibintangi oleh Gina Degeneres
SELECT film.title, category.name AS category
    FROM film
    JOIN film_actor
    ON film.film_id = film_actor.film_id
    JOIN film_category
    ON film_actor.film_id = film_category.film_id
    JOIN category
    ON film_category.category_id = category.category_id
    WHERE film_actor.actor_id = 107 AND category.name = 'sci-fi';
