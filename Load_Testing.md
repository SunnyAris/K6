Dokumentation
https://k6.io/docs/using-k6/http-requests/

## Test 1
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

## Test 2 

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

## Test 3 

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


## Test 4


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

## Test 5


```
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    scenarios: {
        authorization: {
            executor: 'ramping-vus',
            exec: "startAuthorization",
            startVUs: 0,
            stages: [
                { duration: '1s', target: 20 },
                { duration: '10s', target: 0 }
            ]
        },

        users: {
            executor: 'per-vu-iterations',
            exec: "getUsers",
            vus: 30,
            iterations: 2,
            startTime: '5s'
           }
        },

        thresholds: {
            http_req_duration: ['p(95) < 400'],
            http_req_failed: [`rate<=0.05`]
        }
    }

    export function startAuthorization () {
        const url = 'https://httpbin.org/post';
        const payload = JSON.stringify({
            login: "test",
            password: "password"
    
        })

        const params = {
            headers: {
                'Content-Type': 'application/json'
            }
        }

        const res = http.post(url, payload, params);

        check(res, {
            'status code is 200': (r) => r.status == 200,
            'body include login': (r) => r.body.includes("test")
        })

        sleep(1)
    }

    export function getUsers() {
        const res = http.get('http://jsonplaceholder.typicode.com/users')
        check(res, {
            'status code is 200': (r) => r.status == 200,
            'body is includes': (r) => r.body.length > 0
        })
        sleep(1)
    }
```



## `scenarios` - tworzymy scenariusz w oparciu o skrypt testowy.

Określamy dwa scenariusze, które będziemy sprawdzać 
- authorization
- users 
  
## 1 scenariusz

```
 scenarios: {
        authorization: {
            executor: 'ramping-vus',
            exec: "startAuthorization",
            startVUs: 0,
            stages: [
                { duration: '1s', target: 20 },
                { duration: '10s', target: 0 }
            ]
        },

```

`authorization` (logowanie)

`executor: 'ramping-vus'` 
(typ silnika, który wykożystujemy w testach obciążeniowych) 

Ten silnik pozwala na podzielenie testów obciążeniowych na stages.  Stopniowe zapełnienie serwer daną liczbą użytkowników i zmienia ją zależnie od poszczególnego stagea testu. 

`exec: "startAuthorization"`

`exec` - referencja do funkcji, która ma się wywołać 

`"startAuthorization"` - ta funkcja symuluje uzyskanie JWT tokena

```
export function startAuthorization () {
        const url = 'https://httpbin.org/post';
        const payload = JSON.stringify({
            login: "test",
            password: "password" 

 ```    


`stages` pokazują jak będą wyglądała testy obciążeniowe

 Przez 1s aplikacja zapełnia się 20 użytkownikami po czym przez kolejne 10s spada do 0.

```
 
          { duration: '1s', target: 20 },
          { duration: '10s', target: 0 }

```


## 2 scenariusz 

```
users: {
            executor: 'per-vu-iterations',
            exec: "getUsers",
            vus: 30,
            iterations: 2,
            startTime: '5s'
           }
        },

```

`users`
(wyświetlenie listy użytkowników, sprwadzenie ilu jest dostępnych itd.)


`executor: 'per-vu-iterations'`

Daje możliwość napełnienia serwera np. 2000 użytkownikami natychmiast. W różnych iteraciach. Określamy ilość iteracji w jakich na serwer ma wejść dana ilość użytkowników.

Iteracja – czynność powtarzania tej samej operacji w pętli z góry określoną liczbę razy lub aż do spełnienia określonego warunku.


`exec: "getUsers"`

`exec` - referencja do funkcji, która ma się wywołać 

`"getUsers"` - robi get request żeby sprawdzić jacy użytkownicy są dostępni w bazie 


```
export function getUsers() {
        const res = http.get('http://jsonplaceholder.typicode.com/users')
```


Będą 2 iteracje i w każdej iteracji będzie 30 użytkowników na raz. 

```
            vus: 30,
            iterations: 2,


```


`startTime` - opóżnienie startu czyli dopiero po 5 sekundach od uruchomienia się skryptu wykona się ten scenariusz. 
```
           startTime: '5s'

```


## `thresholds`
```
thresholds: {
            http_req_duration: ['p(95) < 400'],
            http_req_failed: [`rate<=0.05`]
        }
    }
```

`thresholds` - pozwala zdefiniować globalne warunki 
```
http_req_duration: ['p(95) < 400'] 
```
95% testów muszą wykonać się w czasie 400 ms

```
http_req_failed: [`rate<=0.05`]
```
Odsetek niepowodzeń będzie mniejszy bądź równy 0.05%

## Zrobienie zaptania `POST` gdzie przekazywane są w formacie JSON jakieś `body`. 
W tym wypadku jest to POST 

login: "test",

 password: "password"

```
export function startAuthorization () {
        const url = 'https://httpbin.org/post';
        const payload = JSON.stringify({
            login: "test",
            password: "password"
    
        })
```
## Tworzony jest parametr przekazujący `headers`

```
const params = {
            headers: {
                'Content-Type': 'application/json'
            }
        }
```
## Weryfikacja czy status code jest 200 i czy `body` zawiera "test"
```
check(res, {
            'status code is 200': (r) => r.status == 200,
            'body include login': (r) => r.body.includes("test")
        })
```
![Alt text](<test 5a.png>)

## Zapytanie `GET`, które zwraca jako zasób listę wszystkich użytkowników 
Sprawdza bazę danych użytkowników czy status code jest 200 i czy body lenght obiekty jest większe od 0 czyli czy jakieś dane zostały zwrócone 
```
export function getUsers() {
        const res = http.get('http://jsonplaceholder.typicode.com/users')
        check(res, {
            'status code is 200': (r) => r.status == 200,
            'body is includes': (r) => r.body.length > 0
        })
```

## Rezultat testu obciążeniowego


![Alt text](<test 5.png>)
