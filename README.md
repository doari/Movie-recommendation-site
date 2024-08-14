# Jayflix - 영화 추천 시스템

이 프로젝트는 사용자가 좋아하는 영화를 선택하면 해당 영화와 유사한 영화를 추천해주는 간단한 영화 추천 시스템입니다. 이 시스템은 콘텐츠 기반 필터링(Content-Based Filtering) 기법을 사용하여 영화 간 유사도를 계산하며, TMDb API를 통해 영화의 포스터와 제목 정보를 가져옵니다.

## 주요 기능

- **영화 선택**: 사용자는 제공된 영화 목록에서 자신이 좋아하는 영화를 선택할 수 있습니다.
- **영화 추천**: 선택한 영화와 유사한 10개의 영화를 추천해 줍니다.
- **포스터 및 제목 표시**: 추천된 영화의 포스터와 제목을 화면에 표시합니다.

## 코드 설명

### 1. 라이브러리 및 API 설정

프로젝트에서는 `pickle`을 사용하여 미리 계산된 영화 데이터와 코사인 유사도 행렬을 불러옵니다. 또한, `tmdbv3api` 라이브러리를 사용하여 TMDb API를 통해 영화 정보를 가져옵니다.

```python
import pickle
import streamlit as st
from tmdbv3api import Movie, TMDb
```

### 2. TMDb API 키 설정
TMDb API를 사용하기 위해 API 키와 언어 설정을 합니다. 여기서는 한국어(Korean)로 설정하였습니다.

```python

tmdb = TMDb()
tmdb.api_key = 'your_api_key_here'  # 본인의 TMDb API 키로 대체
tmdb.language = 'ko-KR'
movie = Movie()
```
### 3. 영화 추천 함수
`get_recommendations` 함수는 선택한 영화와 유사한 영화를 추천하는 핵심 로직을 담당합니다.

- **영화 인덱스 추출**: 선택한 영화의 인덱스를 찾습니다.
- **유사 영화 계산**: 코사인 유사도를 기준으로 유사한 영화를 계산하고, 유사도가 높은 순서로 정렬합니다.
- **영화 정보 추출**: 유사한 영화의 포스터와 제목을 TMDb API를 통해 가져옵니다.

```python
def get_recommendations(title):
    idx = movies[movies['title'] == title].index[0]
    sim_scores = list(enumerate(cosine_sim[idx]))
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    sim_scores = sim_scores[1:11]  # 자기 자신을 제외한 10개의 추천 영화

    movie_indices = [i[0] for i in sim_scores]
    
    images, titles = [], []
    for i in movie_indices:
        id = movies['id'].iloc[i]
        details = movie.details(id)

        image_path = details['poster_path']
        if image_path:
            image_path = 'https://image.tmdb.org/t/p/w500' + image_path
        else:
            image_path = 'no_image.jpg'

        images.append(image_path)
        titles.append(details['title'])
    
    return images, titles
```

### 4. 데이터 로드 및 웹 인터페이스 구성
`pickle`로 저장된 영화 데이터와 코사인 유사도 행렬을 로드하고, Streamlit을 사용하여 웹 인터페이스를 구성합니다. 사용자는 제공된 영화 목록에서 좋아하는 영화를 선택하고, "추천 영화 조회" 버튼을 눌러 추천 영화를 확인할 수 있습니다.

```python
movies = pickle.load(open('movies.pickle', 'rb'))
cosine_sim = pickle.load(open('cosine_sim.pickle', 'rb'))

st.set_page_config(layout='wide')
st.header('Jayflix')

movie_list = movies['title'].values
title = st.selectbox('좋아하는 영화를 검색해주세요.', movie_list)

if st.button('추천 영화 조회'):
    with st.spinner('잠시만 기다려 주세요...'):
        images, titles = get_recommendations(title)

        idx = 0
        for i in range(0, 2):
            cols = st.columns(5)
            for col in cols:
                col.image(images[idx])
                col.write(titles[idx])
                idx += 1
```
## 사용 방법
1. 코드를 실행하면 웹 페이지가 열립니다.
2. 드롭다운 메뉴에서 좋아하는 영화를 선택합니다.
3. "추천 영화 조회" 버튼을 클릭하면, 해당 영화와 유사한 10개의 영화가 화면에 표시됩니다.

## 요구 사항
- Python 3.x
- **streamlit**, **pickle**, **tmdbv3api**, **pandas** 등의 라이브러리 설치 필요

## 주의사항
TMDb API를 사용하기 위해서는 개인 API 키가 필요합니다. API 키는 `TMDb 웹사이트`에서 무료로 발급받을 수 있습니다.
