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

![Figure 1](images/WebScrapingProject/Pic1.png)

## Exploratory Data Analysis
```{python}
#Distribution of movies over time
distofmovies_plot = movies.groupby('Decade')['Title'].count().plot(kind = 'bar', title = 'Distribution of Movies Over Decades')
distofmovies_plot
```

Words

![Figure 2](images/WebScrapingProject/Pic2.png)


```{python}
#Histogram of Critic Score and Score
movies['CriticScore'] = movies['Metascore']/10
plt.hist(movies['Score'], alpha = .5, bins = 10, color= 'blue', label =  'Score')
plt.hist(movies['CriticScore'], alpha = .5, bins = 10, color = 'pink', label = 'Critic Score')
plt.legend(loc = 'upper right')
plt.title('Histogram of Critic Score and Score')
plt.show()
```

Words

![Figure 3](images/WebScrapingProject/Pic3.png)

```{python}
movies['Genre'] = movies['Genre'].str.split(', ')
movies = movies.explode('Genre')

# Calculate the mean Score and Revenue by Genre
genre_stats = movies.groupby('Genre')['Score', 'Revenue'].mean().sort_values('Revenue', ascending=False).head(10)

# Plotting
fig, ax = plt.subplots(figsize=(14, 8))

# Plot average revenue by genre
genre_stats['Revenue'].plot(kind='bar', color='blue', ax=ax, width=0.4, position=1, label='Average Revenue (in millions)')

# Plot average score by genre on a secondary axis
ax2 = ax.twinx()
genre_stats['Score'].plot(kind='bar', color='orange', ax=ax2, width=0.4, position=0, label='Average Score')

# Set the labels and titles
ax.set_ylabel('Average Revenue (in millions)', color='blue')
ax2.set_ylabel('Average Score', color='orange')
ax.set_xlabel('Genre')
ax.set_title('Average Score and Revenue by Genre')

# Set legends
ax.legend(loc='upper left')
ax2.legend(loc='upper right')

plt.show()
```

Words

![Figure 4](images/WebScrapingProject/Pic4.png)

```{python}
correlation_matrix = movies[['Score', 'Metascore', 'Vote', 'Runtime', 'Revenue']].corr()
plt.figure(figsize=(10, 8))
sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm')
plt.title('Correlation Heatmap of Numerical Features')
plt.show()
```

Words

![Figure 5](images/WebScrapingProject/Pic5.png)

```{python}
average_revenue_by_director = movies.groupby('Director')['Revenue'].mean().nlargest(5)
plt.figure(figsize=(10, 5))
average_revenue_by_director.plot(kind='bar')
plt.title('Top 5 Directors with the Highest Average Revenue')
plt.ylabel('Average Revenue (in millions)')
plt.show()
```

Words

![Figure 6](images/WebScrapingProject/Pic6.png)

## Bivariate Analysis
```{python}
plt.figure(figsize=(14, 7))
sns.scatterplot(data=movies, x='Runtime', y='Score', alpha=0.6, color='red')
plt.title('Scatter Plot of Runtime vs. IMDB Score')
plt.xlabel('Runtime (minutes)')
plt.ylabel('IMDB Score')
plt.show()
```

Words

![Figure 7](images/WebScrapingProject/Pic7.png)

```{python}
plt.figure(figsize=(14, 7))
sns.scatterplot(data=movies, x='Year', y='Score', alpha=0.6, color='green')
plt.title('Scatter Plot of Year of Release vs. IMDB Score')
plt.xlabel('Year of Release')
plt.ylabel('IMDB Score')
plt.show()
```

Words

![Figure 8](images/WebScrapingProject/Pic8.png)

## Text Analysis
```{python}
# creates a string with all the element of the password column:
text = " ".join(str(i) for i in genre1_df.Title)
# lower max_font_size, change the maximum number of word and lighten the background and  generate a word cloud image
wordcloud = WordCloud(background_color="white").generate(text)
# figsize=(10,10), changing (x,y) will change the word cloud
plt.figure(figsize=(10,10))
# Display the generated image:
# the matplotlib way:
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```

![Figure 9](images/WebScrapingProject/Pic9.png)

![Figure 10](images/WebScrapingProject/pic10.png)

![Figure 11](images/WebScrapingProject/pic11.png)

```{python}
avg_runtime_by_length = movies.groupby('Description_Length_Bins')['Runtime'].mean()

plt.figure(figsize=(10, 6))
avg_runtime_by_length.plot(kind='bar', color='blue', alpha=0.7)
plt.title('Average Revenue by Description Length')
plt.xlabel('Description Length (in bins)')
plt.ylabel('Average Revenue')
plt.xticks(rotation=45)
plt.show()
```

![Figure 12](images/WebScrapingProject/pic12.png)

## Comparative Analysis
```{python}
# Top 5 directors in the 1990s
nineties_directors = movies[movies['Decade'] == 1990]['Director'].value_counts().nlargest(5)

# Top 5 directors in the 2010s
twenty_tens_directors = movies[movies['Decade'] == 2010]['Director'].value_counts().nlargest(5)

print("Top 5 directors in the 1990s:")
print(nineties_directors)

print("\nTop 5 directors in the 2010s:")
print(twenty_tens_directors)
```

![Figure 13](images/WebScrapingProject/pic13.png)

```{python}
# Top 5 directors and their highest Metascores in the 1990s
top_directors_90s = movies[movies['Decade'] == 1990].groupby('Director')['Metascore'].max().nlargest(5)
print("Top 5 directors and their highest Metascores in the 1990s:")
print(top_directors_90s)

# Top 5 directors and their highest Metascores in the 2010s
top_directors_2010s = movies[movies['Decade'] == 2010].groupby('Director')['Metascore'].max().nlargest(5)
print("\nTop 5 directors and their highest Metascores in the 2010s:")
print(top_directors_2010s)
```

![Figure 14](images/WebScrapingProject/pic14.png)

```{python}
# Box plot for Revenue in the 1990s
plt.figure(figsize=(8, 6))
sns.boxplot(x='Decade', y='Revenue', data = nineties)
plt.title('Revenue Distribution in the 1990s')
plt.xlabel('Decade')
plt.ylabel('Revenue')
plt.show()

# Box plot for Revenue in the 2010s
plt.figure(figsize=(8, 6))
sns.boxplot(x='Decade', y='Revenue', data=twenty_tens)
plt.title('Revenue Distribution in the 2010s')
plt.xlabel('Decade')
plt.ylabel('Revenue')
plt.show()
```

![Figure 15](images/WebScrapingProject/pic15.png)

![Figure 16](images/WebScrapingProject/pic16.png)

```{python}
# plot for 90's
plt.figure(figsize=(8, 6))
plt.scatter(nineties['Revenue'], nineties['CriticScore'], alpha=0.5)
plt.title('Revenue vs CriticScore in the 1990s')
plt.xlabel('Revenue')
plt.ylabel('CriticScore')
plt.grid(True)
plt.show()

# plot for the 2010s
plt.figure(figsize=(8, 6))
plt.scatter(twenty_tens['Revenue'], twenty_tens['CriticScore'], alpha=0.5)
plt.title('Revenue vs CriticScore in the 2010s')
plt.xlabel('Revenue')
plt.ylabel('CriticScore')
plt.grid(True)
plt.show()
```

![Figure 17](images/WebScrapingProject/pic17.png)

![Figure 18](images/WebScrapingProject/pic18.png)

```{python}
# Extract movies from the 1990s
nineties = movies[movies['Decade'] == 1990]

# Extract movies from the 2010s
twenty_tens = movies[movies['Decade'] == 2010]

nineties_top_genres = nineties['Genre'].value_counts().nlargest(10)
twenty_tens_top_genres = twenty_tens['Genre'].value_counts().nlargest(10)

# Plotting top 10 genre distribution
plt.figure(figsize=(10, 6))
plt.bar(nineties_top_genres.index, nineties_top_genres.values, alpha=0.7, label='1990s')
plt.bar(twenty_tens_top_genres.index, twenty_tens_top_genres.values, alpha=0.7, label='2010s')
plt.xlabel('Genres')
plt.ylabel('Frequency')
plt.title('Top 10 Genre Distribution Comparison')
plt.legend()
plt.xticks(rotation=45)
plt.show()
```

![Figure 19](images/WebScrapingProject/pic19.png)

## Multivariate Analysis
```{python}
movies['Title_Word_Count'] = movies['Title'].apply(lambda x: len(x.split()))
movies['Description_Word_Count'] = movies['Description'].apply(lambda x: len(x.split()))
plt.figure(figsize=(15, 10))

# Number of words in a title vs Revenue
plt.subplot(2, 3, 1)
sns.regplot(x='Title_Word_Count', y='Revenue', data=movies)
plt.title('Title Word Count vs Revenue')

# Number of Words in the Description vs Revenue
plt.subplot(2, 3, 2)
sns.regplot(x='Description_Word_Count', y='Revenue', data=movies)
plt.title('Description Word Count vs Revenue')

# Score vs Revenue
plt.subplot(2, 3, 3)
sns.regplot(x='Score', y='Revenue', data=movies)
plt.title('IMDB Score vs Revenue')

# Metascore vs Revenue
plt.subplot(2, 3, 4)
sns.regplot(x='Metascore', y='Revenue', data=movies)
plt.title('Metascore vs Revenue')

# Vote vs Revenue
plt.subplot(2, 3, 5)
sns.regplot(x='Vote', y='Revenue', data=movies)
plt.title('Vote Count vs Revenue')

# Runtime vs Revenue
plt.subplot(2, 3, 6)
sns.regplot(x='Runtime', y='Revenue', data=movies)
plt.title('Runtime vs Revenue')

# Adjust layout
plt.tight_layout()
plt.show()
```

![Figure 20](images/WebScrapingProject/pic20.png)
