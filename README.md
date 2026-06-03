# Sprawozdanie - Laboratorium 12

**Autor:** Roman Rybak

## Cel zadania
Uruchomienie trzech kontenerów `nginx` (`web1`, `web2`, `web3`) w dedykowanej sieci mostkowej `lab12net`, udostępnienie ich na portach hosta, podłączenie wspólnej strony HTML w trybie tylko do odczytu (`ro`) oraz konfiguracja osobnych katalogów na logi za pomocą wolumenów typu bind mount.

---

## Przebieg realizacji zadania 

<img width="1235" height="489" alt="image" src="https://github.com/user-attachments/assets/16010605-f570-4162-bdcd-10217bbb99ca" />


### 1. Utworzenie sieci mostkowej `lab12net`
Zgodnie z poleceniem, kontenery muszą komunikować się w izolowanej sieci definiowanej przez użytkownika.
```bash
docker network create --driver bridge lab12net
c4ed058b0ba70cd0c7f087883d36b8958b3e58940add8a356b4e1c0e66f11a3a
```

### 2. Utworzenie struktury katalogów i pliku HTML
Zgodnie z zasadami używania bind mount, katalogi w systemie macierzystym muszą zostać utworzone przed uruchomieniem kontenerów. Utworzono również plik index.html.
```
mkdir -p ~/lab12/html
mkdir -p ~/lab12/logs/web1
mkdir -p ~/lab12/logs/web2
mkdir -p ~/lab12/logs/web3
echo "<h1>Laboratorium 12</h1><h2>Roman Rybak</h2>" > ~/lab12/html/index.html
```

### 3. Uruchomienie kontenerów Nginx
Trzy kontenery zostały uruchomione z mapowaniem portów (8081, 8082, 8083). Plik HTML został podłączony z uprawnieniami read-only (:ro), a katalogi na logi zostały poprawnie zmapowane do /var/log/nginx.
```
docker run -d --name web1 --network lab12net -p 8081:80 -v ~/lab12/html:/usr/share/nginx/html:ro -v ~/lab12/logs/web1:/var/log/nginx nginx:latest
docker run -d --name web2 --network lab12net -p 8082:80 -v ~/lab12/html:/usr/share/nginx/html:ro -v ~/lab12/logs/web2:/var/log/nginx nginx:latest
docker run -d --name web3 --network lab12net -p 8083:80 -v ~/lab12/html:/usr/share/nginx/html:ro -v ~/lab12/logs/web3:/var/log/nginx nginx:latest
```
<img width="1268" height="237" alt="image" src="https://github.com/user-attachments/assets/fbeca868-22b8-4117-8565-7752293cee99" />


### 4. Weryfikacja działania środowiska

<img width="804" height="314" alt="image" src="https://github.com/user-attachments/assets/1d89f06d-67fd-43b3-98c8-7372035e02d5" />

Aby dowieść, że serwery działają poprawnie, wysłano zapytania HTTP do wszystkich trzech kontenerów za pomocą narzędzia curl. Następnie sprawdzono, czy logi z żądań zapisały się na dysku lokalnym.
```
curl localhost:8081
<h1>Laboratorium 12</h1><h2>Roman Rybak</h2>
cat ~/lab12/logs/web1/access.log
172.21.0.1 - - [03/Jun/2026:20:21:43 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"

curl localhost:8082
<h1>Laboratorium 12</h1><h2>Roman Rybak</h2>
cat ~/lab12/logs/web2/access.log
172.21.0.1 - - [03/Jun/2026:20:26:14 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"

curl localhost:8083
<h1>Laboratorium 12</h1><h2>Roman Rybak</h2>
cat ~/lab12/logs/web3/access.log
172.21.0.1 - - [03/Jun/2026:20:26:34 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"

curl localhost:8081
<h1>Laboratorium 12</h1><h2>Roman Rybak</h2>
cat ~/lab12/logs/web1/access.log
172.21.0.1 - - [03/Jun/2026:20:21:43 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"
172.21.0.1 - - [03/Jun/2026:20:26:51 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"
```

### 5. Weryfikacja trwałości danych
Aby udowodnić niezależność logów od cyklu życia kontenera, przeprowadzono test polegający na usunięciu kontenera web1 i sprawdzeniu dostępności pliku na hoście:

1) Wymuszone usunięcie kontenera web1:
```
docker rm -f web1
web1
```
2) Weryfikacja obecności logów w systemie macierzystym po całkowitym zniszczeniu kontenera:
```
cat ~/lab12/logs/web1/access.log
172.21.0.1 - - [03/Jun/2026:20:21:43 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"
172.21.0.1 - - [03/Jun/2026:20:26:51 +0000] "GET / HTTP/1.1" 200 45 "-" "curl/8.18.0" "-"
```
Mimo trwałego zniszczenia środowiska kontenerowego, logi pozostały całkowicie bezpieczne i nienaruszone na dysku hosta, co udowadnia poprawną konfigurację wolumenu.

<img width="702" height="187" alt="image" src="https://github.com/user-attachments/assets/1140dcb9-a22d-4253-8980-87ad11393913" />

Podsumowanie
Wszystkie warunki zadania zostały w pełni zrealizowane. Kontenery pomyślnie odbierają ruch z sieci zewnętrznej za pomocą zdefiniowanych portów, współdzielą jeden plik HTML oraz zapisują logi do wskazanych wolumenów na lokalnym dysku komputera, zapewniając trwałość danych.
