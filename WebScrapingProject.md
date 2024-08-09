#### [<back to projects](./projects.md)
# Web Scraping in Python using Beautiful Soup
## Introduction
blah blah

## Scraping
```{python}
url='https://www.imdb.com/search/title/?sort=user_rating,desc&groups=top_1000' #'https://www.imdb.com'#
page=requests.get(url,headers=HEADERS)
page

HTTPStatus(page.status_code).phrase

soup=BeautifulSoup(page.content,'html')

soup.prettify()

dataDict1 = {
    'Title': [],
    'Year Released, Movie Length, Rating': [],
    'Review Score': [],
    'Description': [],
    'Vote': [],
    'Metascore': []
}

movieCards=soup.find_all('div',class_='sc-53c98e73-4 gOfInm dli-parent')
for card_content in movieCards:
    dataDict1['Title'].append(card_content.find('div',class_="ipc-title ipc-title--base ipc-title--title ipc-title-link-no-icon ipc-title--on-textPrimary sc-43986a27-9 gaoUku dli-title").text.strip())
    dataDict1['Year Released, Movie Length, Rating'].append(card_content.find('div',class_='sc-43986a27-7 dBkaPT dli-title-metadata').text.strip())
    dataDict1['Review Score'].append(card_content.find('div',class_='sc-e3e7b191-0 jlKVfJ sc-43986a27-2 bvCMEK dli-ratings-container').text.strip())
    dataDict1['Description'].append(card_content.find('div',class_='ipc-html-content-inner-div').text.strip())
    dataDict1['Vote'].append(card_content.find('div',class_='sc-53c98e73-0 kRnqtn').text.strip())
    metascore_element = card_content.find('span', class_='sc-b0901df4-0 bcQdDJ metacritic-score-box')
    if metascore_element is not None:
      dataDict1['Metascore'].append(metascore_element.text.strip())
    else:
      dataDict1['Metascore'].append('')

def convert_to_minutes(time_str):
    time = time_str.split(' ')
    minutes = 0
    for part in time:
        if 'h' in part:
            minutes += int(part.rstrip('h')) * 60
        elif 'm' in part:
            minutes += int(part.rstrip('m'))
    return minutes

imdb = pd.DataFrame.from_dict(dataDict1)
imdb['Year'] = imdb['Year Released, Movie Length, Rating'].str.extract(r'^(\d{4})')
imdb['Movie Length and Rating'] = imdb['Year Released, Movie Length, Rating'].str[4:]
imdb['Movie Length'] = imdb['Movie Length and Rating'].str.extract(r'(.*m)')
imdb['Rating'] =  imdb['Movie Length and Rating'].str.extract(r'm(.*)')
imdb = imdb.drop('Movie Length and Rating', axis = 1)
imdb = imdb.drop('Year Released, Movie Length, Rating', axis = 1)
imdb['Score'] = imdb['Review Score'].str.extract(r'(\d\.\d)')
imdb['Vote'] = imdb['Vote'].str.replace('Votes', '', regex=False)
imdb['Rank'] = imdb['Title'].str.split('.').str[0]
imdb['Title'] = imdb['Title'].str.replace(r'^\d+\.', '', regex=True)
imdb['Runtime'] = imdb['Movie Length'].apply(convert_to_minutes)
imdb = imdb.drop('Movie Length', axis=1)
imdb = imdb.drop('Review Score', axis=1)
imdb
```

## Exploratory Data Analysis


## Bivariate Analysis


## Text Analysis


## Comparative Analysis


## Multivariate Analysis

