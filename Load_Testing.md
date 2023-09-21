Dokumentation
https://k6.io/docs/using-k6/http-requests/

### Test 1
- Jeden wirtualny użutkownik 
- Test wykonany lokalnie
- Jedna iteracja

```
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
  http.get('https://test.k6.io');
  sleep(1);
}
```

![Alt text](<test 1.png>)

### Test 2 

- 10 wirtualnych użytkowników
- Test wykonany lokalnie
- Jedna iteracja
- Czas trwania testu 5 sekund

```
import http from 'k6/http';
import { sleep } from 'k6';
export const options = {
  vus: 10,
  duration: '5s',
};
export default function () {
  http.get('http://test.k6.io');
  sleep(1);
}

```
![Alt text](<test 2-1.png>)

### Test 3 

`check` - sprawdza czy dane zapytanie zwróciło nam zapytaną wartość 

`stages` - częstotliwości, które określają jak będą wyglądały testy obciążeniowe. 


```
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '20s', target: 20 },
    { duration: '5s', target: 0 },
  ],
};

export default function () {
  const res = http.get('https://httpbin.test.k6.io/');
  check(res, { 'status was 200': (r) => r.status == 200 });
  sleep(1);
}
```

`stages` pokazują jak będą wyglądała testy obciążeniowe

 Przez 20 s aplikacja zapełnia się 20 użytkownikami po czym przez kolejne 5s spada do 0.


Dodatkowo `check` sprawdza czy status tego responsa jest 200

![Alt text](<test 3.png>)


### Test 4


```
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '20s', target: 20 },
    { duration: '5s', target: 0 },
  ],
};

export default function () {
  const res = http.get('https://httpbin.test.k6.io/');
  check(res, { 'status was 200': (r) => r.status == 500 });
  sleep(1);
}
```

Zmieniając 200 na 500 możeby sprawdzić czy status code dla każdego zapytania jest 500

W tym przypdaku wyskoczy błąd.

![Alt text](<test 4.png>)








