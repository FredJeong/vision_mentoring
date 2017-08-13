# 비전멘토링 접수 페이지

## 학교 목록 갱신

#### Prerequesites
* Anaconda with Python 3 / or Python 3
* Pandas가 설치되어있어야 함. (`pip install pandas`)
* Python 패키지를 설치할 수 있는 정도의 능력

1. 교육통계서비스 (http://kess.kedi.re.kr/index) 에 들어가서 알림,서비스 -> 자료실 -> `xxxx년 유초중등 및 고등교육기관 주소록` 처럼 써있는 파일을 찾는다.
2. 다운받은 후 `고등학교 목록.xlsx`를 csv파일로 저장한다. 파일 이름은 `list.csv`. 제목이 있는 행 위쪽 행들은 날린다.

#### list.csv 예시
```
연도,유형구분,세부유형,시도,행정구,학교명,본분교,학교상태,설립,남여공학,우편번호,도로명주소,연락처,팩스번호,홈페이지,
2016,특성화고(직업),상업고등학교,서울,종로구,경기상업고등학교,본교,기존(원)교,공립,남여공학,030-47 ,서울특별시 종로구 자하문로 136 . 경기상업고등학교 (청운동),02-737-6490,02-722-0504,http://www.ggc.hs.kr,
2016,일반고,일반고등학교,서울,종로구,경복고등학교,본교,기존(원)교,공립,남자,030-47 ,서울특별시 종로구 자하문로28가길 9 . 경복고등학교 (청운동),02-737-4471,02-736-0422,http://www.kyungbock.hs.kr,
2016,일반고,일반고등학교,서울,종로구,경신고등학교,본교,기존(원)교,사립,남자,030-67 ,서울특별시 종로구 혜화로 74 (혜화동.경신중고등학교),02-762-0393,02-762-0387,http://kyungshin.hs.kr,
```

3. [Anaconda가 설치되어있을 경우] `VM_data.ipynb` 노트북을 실행 후 학교 목록 업데이트 코드를 실행한다. 혹시 또 이상한 행정구역이 생길 경우, 아래의 도시 업데이트 코드를 먼저 실행해서 cities.json도 업데이트한다.
4. 그렇지 않을 경우 아래 코드를 실행시킨다.

학교 목록 저장 코드
```python
import csv
from collections import defaultdict
import json

with open('cities.json', 'r') as cityfile:
    cities = json.loads(cityfile.read())

code_ref = defaultdict(dict)

for city in cities:
    code_ref[city['province']][city['city']] = city['city_id']

keys = ('시도','행정구','학교명','도로명주소')

schools = []
name_collection = defaultdict(list)


with open('list.csv', 'r') as csvfile:
    reader = csv.reader(csvfile, delimiter=',')
    title = next(reader)
    PROV = title.index(keys[0])
    CITY = title.index(keys[1])
    NAME = title.index(keys[2])
    ADDR = title.index(keys[3])

    for row in reader:
        schools.append({
            'city_id': code_ref[row[PROV]][row[CITY]],
            'count': 0,
            'name': row[NAME],
            'id': reader.line_num - 2
        })
        name_collection[row[NAME]].append(len(schools)-1)

for k, v in name_collection.items():
    if len(v) > 1:
        print(k, '(%d)' % len(v), end=': ')
        for i in v:
            schools[i]['name'] += ' (%s)' % (cities[schools[i]['city_id']]['city'])
            print (schools[i]['name'], end=', ')
        print()

with open('schools.json', 'w') as schoolfile:
    json.dump(schools, schoolfile, ensure_ascii=False, indent=2)
```

도시 목록 저장 코드
```python
#recover city list

import csv
from collections import defaultdict
import json

keys = ('시도','행정구','학교명')


cities = []

with open('list.csv', 'r') as csvfile:
    reader = csv.reader(csvfile, delimiter=',')
    title = next(reader)
    PROV = title.index(keys[0])
    CITY = title.index(keys[1])
    for row in reader:
        district = '%s/%s' % (row[PROV], row[CITY])
        if district not in cities:
            cities.append(district)
for i in range(len(cities)):
    info = cities[i].split('/')
    cities[i] = {
        'city_id': i,
        'city': info[1],
        'province': info[0]
    }

with open('cities.json', 'w') as cityfile:
    json.dump(cities, cityfile, ensure_ascii=False, indent=2)
```

5. Firebase 콘솔에서 데이터베이스 창으로 간 후, school을 누르고 오른쪽 위에 목록 버튼 -> JSON 가져오기
6. 새로 생성된 `schools.json` 파일을 업로드한다.
7. 행정구역 목록을 업데이트했다면, cities에 대해 동일한 작업을 수행한다.

## 학생 목록 가져오기

`VM_data.ipynb` 노트북 실행 후, `The content below is real code that generates list of participants` 섹션의 코드를 실행한다.
