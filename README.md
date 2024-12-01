# 💻빅데이터분석 프로젝트 [서울시 침수피해 분석]

## 1️⃣ 주제 선정 배경
- 매년 7~9월 집중 호우, 태풍의 이유로 다수의 침수피해가 발생
- 서울은 뉴스에서도 잦은 빈도로 침수피해 소식을 확인 가능(ex. 강남)
- [서울시 침수피해 뉴스 빈도](https://github.com/SolarHO/FloodDamage_BigData/blob/main/%ED%99%8D%EC%88%98%ED%94%BC%ED%95%B4_%EB%89%B4%EC%8A%A4_%EB%B9%88%EB%8F%84_%EB%B6%84%EC%84%9D.ipynb)
  
  ![image](https://github.com/user-attachments/assets/bfeb1bcc-c7f6-449d-a884-264b92db0775)

> 데이터 출처: 환경 빅데이터 플랫폼 기상재해 뉴스 데이터<br>(https://www.bigdata-environment.kr/user/data_market/detail.do?id=9ffcaf60-9771-11ee-a443-a7e161ec5b2c)

<hr>

## 2️⃣ 침수피해 시각화

- [행정안전부 생활안전지도 API에서 도시침수지도와 침수흔적도 shp파일을 추출](https://github.com/SolarHO/FloodDamage_BigData/blob/main/%EB%8F%84%EC%8B%9C%EC%B9%A8%EC%88%98%EC%A7%80%EB%8F%84%2C_%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%B6%94%EC%B6%9C.ipynb)
<pre>
  <code>
    import geopandas as gpd
    from requests import Request
    from owslib.wfs import WebFeatureService
    
    url = 'https://geo.safemap.go.kr/geoserver/safemap/wms'
    wfs = WebFeatureService(url=url,version='1.1.0')
    
    wfs.contents
    
    #침수흔적도 추출(A2SM_FLUDMARKS)
    params = dict(service='wfs', version='1.1.1', typeName='A2SM_FLUDMARKS',
                  request='GetFeature', outputFormat='application/json')
    q = Request('GET', url, params=params).prepare().url
    data = gpd.read_file(q)
    
    data.to_file('FloodMap.shp', driver='ESRI Shapefile')
    
    #도시침수지도 추출(A2SM_FLOODDAMAGE_old)
    params = dict(service='wfs', version='1.1.1', typeName='A2SM_FLOODDAMAGE_old',
                  request='GetFeature', outputFormat='application/json')
    q = Request('GET', url, params=params).prepare().url
    data = gpd.read_file(q)
    
    data.to_file('Flooddamage.shp', driver='ESRI Shapefile')
  </code>
</pre>

<hr>

### folium을 이용한 지도 시각화
  기본 지도는 등고선을 직관적으로 보기 위해 stadiamaps의 terrain 지도 사용<br>
  지도 좌표는 서울 시청을 중심으로 설정(37.5665, 126.978)<br>
  지도에 서울 행정구역을 붉은색으로 표시
  
- [도시침수지도](https://github.com/SolarHO/FloodDamage_BigData/tree/main/%EB%8F%84%EC%8B%9C%EC%B9%A8%EC%88%98%EC%A7%80%EB%8F%84)
    - 도시침수 지도는 행정안전부에서 도시 지역에서 발생 가능한 침수 위험을 예측하기 위해 제작된 지도
  <pre> 수집한 도시침수지도shp파일을 geopandas를 이용해 geojson 형식으로 변환 -> geojson파일을 folium을 이용해 지도에 시각화
    -> 침수 범례에 따라 색상으로 구분하여 폴리곤으로 표시
  </pre>
  ![도시침수지도](https://github.com/user-attachments/assets/99016e22-ff16-43c0-85c6-30c430a07d31)
- [침수흔적도](https://github.com/SolarHO/FloodDamage_BigData/tree/main/%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84)
    - 침수흔적도는 과거에 실제로 발생한 침수 피해의 기록을 기반으로 제작된 지도
  <pre> 수집한 침수흔적도shp파일을 geopandas를 이용해 geojson 형식으로 변환 -> geojson파일을 folium을 이용해 지도에 시각화
    -> 침수 범례에 따라 색상으로 구분하여 폴리곤으로 표시
  </pre>
  ![침수흔적도](https://github.com/user-attachments/assets/855e502c-a7e5-4200-a147-01418c3b0cb0)

  > 도시침수지도와 침수흔적도간 데이터 사이에 오차가 크게 발생  
  > 한강 남쪽 강서구와 영등포구 부근에서는 비슷한 분포를 보임

<hr>

### 침수흔적도 요인분석
  침수의 요인에는 기상재해이라는 특성상 다양한 관점에서 접근해야 하지만 크게 침수구역의 저지대 위치와<br>
  침수구역의 불투수 면적이 침수 요인으로 작용한다.
- [서울시 등고선 저지대 분석](https://github.com/SolarHO/FloodDamage_BigData/tree/main/%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84)
    1. 국토교통부 브이월드에서 LARD_ADM_SECT_SGG 파일 다운로드 (https://www.vworld.kr/v4po_main.do)
    2. QGIS 이용 DEM 데이터에서 서울시 행정구역 경계의 등고선을 shp 파일로 추출
  <pre>  서울시 등고선shp 파일을 geopandas를 이용해 geojson 형태로 변환<br> -> 침수흔적도 지도에 등고선 파일을 벡터레이어에 추가
  </pre>
  ![침수흔적도_등고선](https://github.com/user-attachments/assets/03fd1f78-c16c-4a79-a794-3ee78595db2a)
- [서울시 불투수면적](https://github.com/SolarHO/FloodDamage_BigData/tree/main/%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84)
    - 서울 열린데이터 광장에서 서울시 불투수면적 현황 데이터를 다운로드(https://datafile.seoul.go.kr/bigfile/iot/inf/nio_download.do?&useCache=false)
  <pre> 침수흔적도 지도에 '불투수면적 현황csv'파일을 벡터레이어에 추가 -> 자치구별 퍼센트율 사용<br> -> 단계분포도(Choropleth)로 시각화< -> 채우기 컬러는 'YlOrRd'사용
  </pre>
  ![침수흔적도_불투수면적](https://github.com/user-attachments/assets/da2d5608-ab1a-4a20-82b0-dca9bdfbe278)

  >불투수 면적이 상대적으로 많은 영등포구, 동작구, 구로구, 금천구에서 침수피해가 잦음  
  >또한 상대적으로 저지대에 위치한 강서구, 영등포구 부근에서 침수피해가 잦은걸 확인 가능

<hr>

### [침수흔적도 통계](https://github.com/SolarHO/FloodDamage_BigData/blob/main/%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84/%EC%B9%A8%EC%88%98%ED%9D%94%EC%A0%81%EB%8F%84_%ED%86%B5%EA%B3%84.ipynb)
  1. 침수흔적도shp 파일에서 행정구역코드(CTPRVN_CD)가 서울(11)인 데이터만 필터링
  2. 서울시 시군구 코드(SGG_CD)를 구 이름(SGG_Name)으로 변환
  <pre>
    <code>
        # 서울시 시군구 코드와 명칭 매핑
        sgg_cd_to_name = {
            "11110": "종로구",
            "11140": "중구",
            ...
        }
        
        # SGG_CD를 구 이름으로 변환
        seoul_data["SGG_Name"] = seoul_data["SGG_CD"].map(sgg_cd_to_name)
        
        # 구별 침수 피해 발생 건수 계산
        seoul_damage_count = seoul_data["SGG_Name"].value_counts()
    </code>
  </pre>
  - __구 별 침수피해 발생 건수 비교__
    - 막대그래프(countplot)로 시각화
    ![image](https://github.com/user-attachments/assets/2a552f3b-94c5-404d-89d9-d1b98f7374cc)
    - 상위 10개의 행정구만 원형 그래프로 시각화
    ![image](https://github.com/user-attachments/assets/6f559e2f-f6eb-4bb4-bec1-d042a1cea9e2)

  - __구 별 침수 면적 비교(FLUD_AR)__
    - 막대그래프(countplot)로 시각화
    ![image](https://github.com/user-attachments/assets/00ae501b-f006-4778-866e-dfc16c23e65c)
    - 상위 10개의 행정구만 원형 그래프로 시각화
    ![image](https://github.com/user-attachments/assets/0fb3b705-c595-4ca8-8379-92ab3a3bb8a8)

  - __구 별 평균 침수 심도 비교(FLUD_SHIM)__
    - 막대그래프(countplot)로 시각화
    ![image](https://github.com/user-attachments/assets/20ac1894-434d-4f15-b3a5-564b1cbccc9d)
    - 상위 10개의 행정구만 원형 그래프로 시각화
    ![image](https://github.com/user-attachments/assets/245f9d7a-5aa6-4e0e-8a21-803fc27c3116)

<hr>

### [침수 재산피해 통계](https://github.com/SolarHO/FloodDamage_BigData/blob/main/%EC%84%9C%EC%9A%B8%EC%8B%9C_%EC%B9%A8%EC%88%98_%EC%9E%AC%EC%82%B0%ED%94%BC%ED%95%B4_%ED%86%B5%EA%B3%84.ipynb)
  1. 서울통계통합플랫폼(https://stat.eseoul.go.kr)에서 '자연재해+발생+및+피해+현황_20241110115122.csv'파일 다운로드
  2. 전처리
  <pre>
    <code>
      # 칼럼 새로 지정
      df.columns = [
          "자치구별(1)", "자치구별(2)", "사망및실종(명)", "부상(명)", "이재민(명)",
          "주택침수세대(세대)", "전체피해액(천원)", "건물피해액(천원)", "선박피해액(천원)",
          "농경지피해액(천원)", "공공시설피해액(천원)", "사유시설피해액(천원)"
      ]
      
      # 기존 인덱스 제거 및 불필요한 행 제거
      df = df.iloc[2:].reset_index(drop=True)
      
      # "서울시 소계" 행 제거
      df = df[df["자치구별(2)"] != "소계"].reset_index(drop=True)
      print(df)
    </code>
  </pre>
  - 자치구별 주택침수세대 통계
    ![image](https://github.com/user-attachments/assets/92058a5c-e8ac-471a-9470-0e17a0e0ab3c)
  - 자치구별 전체피해액 통계
    ![image](https://github.com/user-attachments/assets/17027a36-7b04-48f5-ab12-c2acfed46f7b)
  - 자치구별 건물&공공시설 피해액 비교
    ![image](https://github.com/user-attachments/assets/c40f15f3-aaab-406d-a9e4-7710be6cce16)
    ![image](https://github.com/user-attachments/assets/f74af704-2528-4b88-94e8-873bd2cac488)

## 3️⃣ 결론
  1. 서울시 침수 피해는 한강 기준으로 남쪽에 위치한 행정구(영등포구, 서초구, 동작구, 구로구 등)가 주로 취약하며
     요인으로는 상대적으로 저지대에 위치함과 불투수 면적에 있다
  2. 침수심도는 여의도 주변을 둘러싼 강서구와 동작구가 가장 크며, 다음으로는 서초구가 있다
  3. 주택침수 피해는 관악구, 영등포구, 동작구가 가장 심하며 이는 반지하 주택의 분포가 영향을 미친 것으로 보인다(추후 상관관계 분석 필요)
  4. 공공시설 피해는 서초구가 가장 심하며 이는 주로 상권가와 지하철역 등 주택보다 공공시설 위주의 분포가 영향을 미친 것으로 보인다(추후 상관관계 분석 필요)

