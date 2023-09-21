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



### Test 2 

- 10 wirtualnych użytkowników
- Test wykonany lokalnie
- Jedna iteracja
- Czas trwania testu 10 sekund

```
k6 run --vus 10 --duration 30s script.js
```




