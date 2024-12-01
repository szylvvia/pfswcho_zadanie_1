# ZADANIE 1 - Programowanie FullStack w Chmurze Obliczeniowej
<div style="text-align: justify;">
W ramach zadania stworzono zestaw plików manifestów (plików yaml) opisujących obiekty środowiska Kubernetes zgodnie z założeniami. Utworzono obiekty zdeklarowane w opracowanych plikach yaml. Potwierdzono ich poprawne uruchomienie za pomocą poleceń.

## Część obowiązkowa

### 0. Utworzenie przestrzeni nazw <i>zad1</i>
W tym celu uruchomiono plik manifestu o nazwie namespace_zad1.yaml za pomocą polecenia ```kubectl apply -f namespace_zad1.yaml```. Aby sprawdzić poprawność wykonanego polecenia wykorzystano  polecenie wyświetajace wszytskie dostępne przestrzenie nazw ```kubectl get namespace```. Na jego podsatwie można wnioskować, że przestrzeń nazw została stworzona i jest aktywna.
<br>

![image](https://github.com/user-attachments/assets/79e5e628-95fb-4d89-acec-a4fa8d56258f)


### 1. Utworzenie ograniczenia na zasoby <i>(quota)</i>
W tym celu uruchomiono plik manifestu o nazwie  quota_zad1.yaml za pomocą polecenia ```kubectl apply -f quota_zad1.yaml```. Obiekt Quota został utworzony wraz z określonymi zasobami:
<ul>
  <li>maksymalna liczba Pod-ów: 10</li>
  <li>dostępne zasoby CPU: 2 CPU (2000m)</li>
  <li>dostępna ilość pamięci RAM: 1,5Gi </li>
</ul>

Aby sprawdzić poprawność utworzonego obiektu wyswietlono najpierw obiekty typu quota z przestrzeni nazw zad1 (ponieważ do tej przestrzeni jest przypisany obiekt) za pomoca polecenia: ```kubectl get quota --namespace=zad1```. Następnie aby rozszerzyć informacje o obiekcie wykorzystano polecenie ```kubectl describe quota zad1quota --namespace=zad1```. Polecenia diagnostyczne pokazują, że ustawione ograniczenia są adekwatne do wymagań zadania. Ilość pamięci RAM została wyrażona jako 1536Mi co jest równoważne 1.5Gi (1.5 * 1024Mi = 1536Mi).
<br>

![image](https://github.com/user-attachments/assets/6a4eca23-4024-4f99-9cd4-7d93ab20586b)

### 2. Utworzenie Pod-a <i>worker</i>
W tym celu uruchomiono plik manifestu o nazwie pod_worker.yaml za pomocą polecenia: ```kubectl apply -f pod_worker.yaml```. Pod został utworzony w przestrzeni nazw zad1 i bazuje na obrazie <i>nginx</i>. Ma następujące ograniczenia na wykorzystywane zasoby:
<ul>
  <li>limits
    <ul>
      <li>memory: 200Mi</li>
      <li>CPU: 200m</li>
    </ul>
  </li>
  <li>requests
    <ul>
      <li>memory: 100Mi</li>
      <li>CPU: 100m</li>
    </ul>
  </li>
</ul>

Pierwszym poleceniem diagnostycznym jest wyświetlenie wszystkich podów w przestrzeni nazw zad1 za pomocą polecenia ``` kubectl get pods --namespace=zad1```. Widoczne jest, że pod został utworzony poprawnie (READY 1/1), a jego status jest RUNNING. 
<br>

![image](https://github.com/user-attachments/assets/7e6fdf7a-e60f-4538-8ba9-97ba909a6ce1)

W celu wyświetlenia bardziej szczegółowych informacji skorzystano z polecenia ``` kubectl describe pod worker --namespace=zad1```. Wynikiem polecenia są dodatkowe informacje o obiekcie w tym obraz, ustawione limity, przestrzeń nazw. Ustawione limity zgadzają się, z wymaganiami zadania. 

![image](https://github.com/user-attachments/assets/e506dfd7-4d1d-4397-ac30-bc3bc695bbaf)

W dalszej części rezultatu tego polecenia można sprawdzić logi, które zawierają informacje o stanie lub zmianach w utworzonym zasobie (np. pobranie obrazu, utworzenie kontenera). Ostatni z nich potwierdza, ze pod został uruchomiony z powodzeniem.
<br>

![image](https://github.com/user-attachments/assets/8f6ce168-87c3-40f4-8cf3-c65602fb422f)

### 3. Modyfikacja pliku php-apache.yaml
Bazując na przykładzie oraz dokumentacji Kubernetes zmodyfikowano plik tak by obiekty Deployment i Service utworzone zostały w przestrzeni nazw zad1. Jednocześnieobiekt Deployment ma mieć następujące ograniczenia na wykorzystywane zasoby:
<ul>
  <li>limits
    <ul>
      <li>memory: 250Mi</li>
      <li>CPU: 250m</li>
    </ul>
  </li>
  <li>requests
    <ul>
      <li>memory: 150Mi</li>
      <li>CPU: 150m</li>
    </ul>
  </li>
</ul>

Gotowy plik manifestu o nazwie php-apache.yaml został uruchomiony za pomocą polecenia:  ```kubectl apply -f php-apache.yaml```. W celu diagnostycznych wykorzytsano polecenia ```kubectl get deployments --namespace=zad1``` wyświetaljący wszytskie obiekty typy Deployment z określonej przestrzeni nazw oraz ```kubectl get services --namespace=zad1```  wyświetaljący wszytskie obiekty typy Sevice z określonej przestrzeni nazw. Obiekt Deployment został stworzony i uruchiony poprawnie (READY 1/1 oraz STATUS: RUNNING). Analogicznie obiekt typu Service również został utworzony.
<br>

![image](https://github.com/user-attachments/assets/d1d57f9a-7873-4c2a-87ea-5b18a011de85)

Aby wyświetlić szczegółowe infromacje o uwtorzonych obiektach wykorzystano kolejno polecenia ```kubectl describe deployment php-apache --namespace=zad1``` oraz ```kubectl describe service php-apache --namespace=zad1```. Dla obiektu typu Deployment można zweryfikować takie informacje jak nazwa, przestrzeń nazw, czy usatwione limity, żądania. Zgadzają się one z wymaganiami zadania. Warto zwrócić również uwagę, na sekcję z Replicas, gdzie wyświetlone są informacje o replikach. W tym przypaadku jest tylko jedna i jednocześnie jest ona dzialajaca poprawnie.
<br>

![image](https://github.com/user-attachments/assets/1f9f40eb-fd76-47f5-bf30-7ed854dce1cb)

Biorąc pod uwagę szczegółowe informacje o obiekcie Sevice można z nich odczytać nazwę, przestrzeń nazw w której został uruchomiony obiekt oraz adresy czy też port.
<br>

![image](https://github.com/user-attachments/assets/7a4ccbe8-1cda-43e5-933f-cc52c7870c19)

### 4. Utworzenie obiektu <i>HorizontalPodAutoscaler</i>
Utworzono plik manifestu definiujący obiekt HorizontalPodAutoscaler, który pozwoli na autoskalowanie wdrożenia (Deployment) php-apache z zastosowaniem następujących parametrów:
<ul>
  <li>minReplicas: 1</li>
  <li>targetCPUUtilizationPercentage: 50</li>
</ul>
Dodatkowo zadanie wymagało określenia parametru <i>maxReplicas</i> tak aby nie przekroczyć parametrów quoty dla przestrzeni nazw <i>zad1</i>. Aby możliwe było przeliczenie tego parametru należy przeanalizować parametry obiektów:
<ul>
  <li>quota
    <ul>
      <li>memory: 1536Mi</li>
      <li>CPU: 2000m</li>
    </ul>
  </li>
  <li>deployment
    <ul>
      <li>memory: 250Mi<</li>
      <li>CPU: 250m</li>
    </ul>
  </li>
</ul>

Biorąc pod uwage parametr memory quota pomieści maksymalnie 6 replik po 250Mi (6*250Mi = 1500Mi). Wartość ta jest również adekwatna dla parametru CPU. Aby nie przekroczyć parametrów quoty dla autoscalera ustawiono wartość <i>maxReplicas: 6</i>. Parametr <i>targetCPUUtilizationPercentage</i> mówi o tym, że jeśli obciążenie CPU wzrośnie powyżej 50%, autoscaler uruchomi dodatkowe pody, aby zrównoważyć obciążenie.
<br>

Gotowy plik manifestu o nazwie hpa_php_apache.yaml został uruchomiony za pomocą polecenia:  ```kubectl apply -f hpa_php_apache.yaml```. W celu diagnostycznych wykorzytsano polecenia ```kubectl get hpa --namespace=zad1``` wyświetaljący wszytskie obiekty typu HorizontalPodAutoscaler z określonej przestrzeni nazw. Rezultatem są informacje, że obiekt HPA został uwtorzony, a jego parametry zgadzają się, z tymi określonymi w zadaniu i przeliczoną wartoscią <i>maxReplicas</i>.
<br>

![image](https://github.com/user-attachments/assets/2cf417cd-7517-4af2-b9a1-4a5e52f2f540)

### 5. Uruchomienie zadeklarowanych obiektów i sprawdzenie poprawności działania
Wszytskie obiekty były uruchamiane na bieżąco a ich poprawnosć weryfikowana za pomoca poleceń w punkach powyżej.

### 6. Uruchomienie aplikacji generującej obciążenie dla aplikacji <i>php-apache</i>
W tym celu wykorzystano plik stworzony plik manifestu o nazwie load_generator.yaml dla aplikacji generujacej obciazenie. Został utworzony na podstawie polecenia z dokumentacjji Kubernetes. Uruchomiono go za pomocą polecenia:  ```kubectl apply -f load_generator.yaml```. 

![image](https://github.com/user-attachments/assets/c36cf1de-44c0-4e24-8903-b202a7d1205b)

Za pomocą poleenia ```kubectl get hpa php-apache --watch --namespace=zad1``` wyświetlane zostają informacje o zasobie HPA w sposób ciągły, monitorując zmiany w tym zasobie. Obiekt HPA automatycznie skaluje obiekt Deployment do odpoiwedniej liczby replik na podstawie metryki CPU. 

![image](https://github.com/user-attachments/assets/b4161dc9-21bf-4dbf-b0f1-3a6e021d7558)

![image](https://github.com/user-attachments/assets/f87bb06c-8a5c-471e-bdb0-306e1688af0e)

Aby potwierdzić, że aplikacja faktycznie generuje obciążenie, sprawdź metryki CPU i pamięci wykorzystano polecenie ```kubectl top pods --namespace=zad1```

![image](https://github.com/user-attachments/assets/f8dcff8e-9ffa-4961-a73a-7ee6ff22471f)

Wykorzystując polecenie ```kubectl describe hpa php-apache --namespace=zad1``` można zobaczyć szczegółowe informacje o obiekcie w tym jak przebiegało skalowanie.

![image](https://github.com/user-attachments/assets/b39396c2-fc56-47a0-9a2f-a05dfcc94d82)

Na podsatwie szczegółowych informacji o obiekcie deployment można zauważyć, ze wszytskie repliki są tworzone poprawnie. Skalowanie przebiega pomyślnie. Obiekt HPA uznaje, że liczba replik 4 (maksymalna jest ustawiona jako 6) jest wystarczająca nawet w przypadku obciązenia CPU na poziomie 160%.

![image](https://github.com/user-attachments/assets/cd3dcd80-7e18-4c64-be14-409504c07a3d)

![image](https://github.com/user-attachments/assets/16fc3006-b2f1-41c3-b5dc-fd3bc27f6836)

Gdy generator obciążenia zostanie wyłaczony, obiekt HPA redukuje liczbę replik, tak aby była właściwa dla bieżącego zużycia.

![image](https://github.com/user-attachments/assets/82bb385d-6617-41a7-8ffa-bc474cfa88e1)

Wykorzystując polecenie ```kubectl describe deployment php-apache --namespace=zad1``` można zobaczyć szczegółowe informacje o obiekcie w tym jak przebiegało skalowanie.

![image](https://github.com/user-attachments/assets/08d37b46-4056-43f2-a43a-6fcbe52fc1ed)

Dobrany parametr <i>maxReplicas: 6</i> dla obiektu HPA nie powoduje przekroczenia zasobów, obiekt działa poprawnie, skalowanie przebiega pomyślnie zarówno w górę jak i w dół.

## Część dodatkowa

### 1. Czy możliwe jest dokonanie aktualizacji aplikacji (np. wersji obrazu kontenera) gdy aplikacja jest pod kontrolą autoskalera HPA?
Tak, możliwe jest przeprowadzenie aktualizacji aplikacji (np. wersji obrazu kontenera) podczas gdy aplikacja jest pod kontrolą autoskalera HPA.

![image](https://github.com/user-attachments/assets/3e855cd1-24bb-4cf1-963a-b95877a1eb40)

### 2. Podczas aktualizacji zawsze będa aktywne 2 pod-y realizujące działanie przykładowej aplikacji, jaki parametr strategii rollingUpdate dobrać aby to zagwarantować?
W tym celu należy skorzystać z parametru <i>maxSurge</i> i ustawic jego wartość na 2. Pole to określa maksymalną liczbę podów, które można utworzyć ponad żądaną liczbę podów. Dzięki temu parametrowi nowe pody są uruchamiane zanim stare pody zostaną usunięte, co zapewnia, że zawsze będą aktywne dwa pody, a aplikacja będzie działać bez pzrestojów. Warto również ustawic parametr <i>maxUnavailable</i> na wartość 0, aby określić maksymalną liczbę podów, które mogą być niedostępne podczas procesu aktualizacji. W tym przypadku wszytskie pody muszą być dostępne. Pody nie zostaną usuniete, dopóki aplikacja nie będzie mieć aktywnych nowych podów. Dzięki temu liczba podów, nigdy nie spadnie poniżej określonej liczby podów 2. Takie podejście zapewnia stałą dostępność podów realizujących działanie aplikacji. Przykładowy kod z sekcją rollingUpdate (na podsatwie doumentacji Kubernetes):
```
strategy:
   type: RollingUpdate
   rollingUpdate:
     maxSurge: 2
     maxUnavailable: 0
```
### 3. Nie zostaną przekroczone parametry wcześniej zdefiniowanej quoty dla przestrzeni zad1, jak należy zmienić ustawienia autoskalera HPA?
Ponieważ dobrany parametr <i>maxReplicas: 6</i> dla autosklaera HPA dla obiektu Deployment <i>php-apache</i> jest maksymalną liczbą podów jakie można uruchomić, aby nie przekroczyć zasobów quoty <i>zad1</i> dla wartości memory i CPU, to w momencie gdy ustawiamy parametr startego rolling update <<i>maxSurge: 2</i> może dojść do sytuacji gdy liczba deployment będzie uruchamiać nowe dodatkowe pody (zgodnie z parametrem <i>maxSurge: 2</i>) oraz utrzymywać stare, co w szczytowym momencie może osiągnąć wartość 8. W takiej sytuacji przekroczone zostaną zdefiniowane parametry quoty. Aby temu zapobiec, należy zmniejszyć watość parametry <i>maxReplicas</i> do 4, dzięki czemu nawet jeśli deployment będzie uruchamiał nowe pody i utrzymywał stare, to będzie ich maksymalnie 6 (<i>maxReplicas</i> + <i>maxSurge</i>), a taka wartość jest maksymalna, ktora nie przekroczy parametrów dla zdefiniowenej quoty <i>zad1</i>. Poniżej kod autoskalera HPA ze omówioną zmianą:
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: php-apache
  namespace: zad1
spec:
  maxReplicas: 4
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  targetCPUUtilizationPercentage: 50
status:
  currentReplicas: 0
  desiredReplicas: 0
```
</div>

## Podsumowanie
Jako opracowanie zadania wykonano zarówno część obowiązkową jak i dodatkową. W ramach części obowiązkowej zadania 1 został utworzony szereg plików manifestów dla obiektów opisanych w wymaganiach zadania. Zdefiniowano dla nich określone parametry. Każdy z nich został uruchomiony, a poprawność jego zadania została zweryfikowana dzięki wykorzytsaniu poleceń diagnostycznych (wszytskie wykorzystane zostały wymienione w sprawozadniu). Dla autosklaera HPA została dobrana wartość parametru <i>maxReplicas</i> zgodnie z nałożonymi ograniczeniami zasóbów quota oraz limitów obiektu Deployment. Uruchomiono generator obciążenia, aby zweryfikować poprwność działania autoskalera, a także wykorzystano do tego polecenia diagnostyczne. Część dodatkowa składa się z odpowiedzi na pytania wraz z wskazaniem żródła, a także zmian w plikach manifestów. Wykonanie zadania zwłaszcza z części obowiązkowej było możliwością zweryfikowania umiejętności w praktycznym zadaniu z zakresu tworzenia różnego rodzaju obiektów. Część nieobowiazkowa pozwoliła na rozszerzenie wiedzy z zakresu aktualizacji aplikacji oraz dobierania parametrów stratego <i>rollingUpdate</i>, tak aby zapewnić ciągłość w dostępie do aplikacji. Wykonanie zadania przebiegło pomyślnie. 

<hr>
<div style="text-align: justify;">
  <i>Opracowanie zadania powstało w ramach laboratorium na Politechnice Lubelskiej.</i>
</div>
