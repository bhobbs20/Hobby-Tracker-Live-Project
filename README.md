# Hobby Tracker Live Project Summary

## Introdduction

For the last two weeks, I have worked with a team of peers at the tech academy developing a full scale MVT Web Application in Python and Django. 

## Back End Stories
* Design Models
* Crud
* API
* WebScraper


## Front End Stories
* forms
* details
* display


### Design Models
This app had two models. A model for Reviews and a Model for games.

#### Game Model

``` 
class Game(models.Model):
    title = models.CharField(max_length=300)..
    platform = models.CharField(max_length=350)..
    summary = models.TextField(max_length=1000)..
    multi_player = models.BooleanField(default=False)
    online_play = models.BooleanField(default=True)
    advised_age = models.IntegerField(default=0)
    cost = models.DecimalField( max_digits=6, decimal_places=2)
    img = models.CharField(max_length=200)
    genre = models.CharField(max_length=200)
```
```
        def __str__(self):
            return self.title
```
#### Review Model

```
class Review(models.Model):
    title = models.CharField(max_length=200)
    game = models.ForeignKey(Game, on_delete=models.CASCADE)
    dads_op = models.TextField(max_length=1000, null=False)
    sons_op = models.TextField(max_length=1000, null=False)
    dad_vote = models.BooleanField(default=None)
    son_vote = models.BooleanField(default=None)
```

If dad and son aggree they both like the game, recommend game.

```
    def is_recommend(self):
        if self.dad_vote and self.son_vote == True:
            return True
        return False
```
```
    def __str__(self):
        return self.title
```

## CRUD Functions

#### Create Game

```
def add_game(request):
    form = GameForm(request.POST or None)  # If posted form, get it
    if form.is_valid():  # Checks for form errors
        form.save()  # Saves form to the db
        return redirect('dsgames')  # Redirects to dsgaming home
    else:
        form = GameForm()  # Creates a new blank form
    return render(request, 'DadSonGaming/Games/dsg_create_game.html', {'form': form}) # renders create page with form context
```

#### Create Review

```
def create_review(request):
    form = ReviewForm(request.POST or None) # If posted form, get it
    if form.is_valid(): # Checks for form errors
        form.save() # Saves form to the db
        return redirect('dsgames') # Redirects to dsgaming home
    else:
        form = ReviewForm()  # Creates a new blank form
    return render(request, 'DadSonGaming/Reviews/dsg_create_review.html', {'form': form}) # renders create page with form context
```

#### Read All Reviews

```
def index(request):
    all_reviews = Review.objects.all() # Gets all Reviews
    # all_games = Game.objects.all() # Gets all Games

    # Put reviews and games into context dictionary
    context = {
        'all_reviews': all_reviews,
        # 'all_games': all_games,
    }

    return render(request, 'DadSonGaming/Reviews/dsg_reviews_index.html', context)
```

#### Read Indvidual Review Details

```
def review_detail(request, pk):
    # PK to int
    pk = int(pk)

    # Gets single review from db or returns 404
    review = get_object_or_404(Review, pk=pk)

    # Review in dictionary context
    context = {
        'review': review
    }

    # renders single review detail page with context
    return render(request,'DadSonGaming/Reviews/dsg_review_detail.html', context)
```

#### Edit Reviews 

```
def edit_review(request, id):
    # id to int
    id = int(id)

    review = get_object_or_404(Review, id=id)
    # review = Review.objects.get(id=id)
    form = ReviewForm(request.POST or None, instance=review)

    # form is valid, save to db, redirect to reviews page
    if form.is_valid():
        form.save()
        return redirect('reviews')

    # Review in dictionary context
    context = {
        'form': form,
    }
    # renders edit form with context
    return render(request, 'DadSonGaming/Reviews/dsg_edit_review.html', context)
```

#### Delete Review


```
def delete_review(request, id):
    # id to int
    id = int(id)
    # Try and Except Error handling, trying it out, Did I do this right?
    try:
        # Get single review
        review = Review.objects.get(id = id)
    # if no review exist throw error object doesnt exist
    except Review.DoesNotExist:
        # redirect to reviews page
        return redirect('reviews')
    # everything ok, delete
    review.delete()
    # redirect to reviews page
    return redirect('reviews')
```

## Video Game API
#### Api Used: https://api.rawg.io/api/games
This code connects to the RAWG API, grabs first 20 games, and returns them in json format
```
def my_game_api():
    # Request for list of games from rawg api
    api_request = requests.get(
        "https://api.rawg.io/api/games?page_size=20",
    )
    # Games in json format
    api_games = json.loads(api_request.content)
    return api_games
```

### API Index Page
Displays the first 20 games from RAWG API

```
def game_api(request):
    # puts games form api into context
    context = {
        'api_games': my_game_api(), # function that makes games api call and gets games
    }
    return render(request, 'DadSonGaming/Games/dsg_game_api.html', context)

```

### Searches for game and displays the game and others alike

```
def search_game_api(request):
    # post request
    if request.method == 'POST':
        # stores users input
        game_item = request.POST['game_item']
        # makes api call and adds users input into search
        game_search_request = requests.get(f"https://rawg.io/api/games?page_size=6&search={game_item}")
        # returns json data of games
        found_game = json.loads(game_search_request.content)

        # prints list of searched games to termianl
        # print(f" here is the game you are looking for:  {found_game} ")

        context = {
            'game_item': game_item,
            'found_game': found_game,
        }

        return render(request, 'DadSonGaming/Games/dsg_search_game.html', context)

    else:
        cant_find_game = "Cant find, Enter a game into form "
        return render(request, 'DadSonGaming/Games/dsg_search_game.html', {'cant_find_game': cant_find_game })
```

## Video Game News Webscraper
### Game News Site Used: https://nichegamer.com/news/

###  Web Scrapper Index Page
Displays new stories with link to story

```
def game_scrapper(request):
    # Gets nichegamer.com/news as html doc
    page = requests.get('https://nichegamer.com/news/')
    # navigatable object
    soup = BS(page.content, 'html.parser')
    # Search for class index-card
    stories = soup.find_all(class_="index-card")
    # Creates empty list
    game_news = []

    # Loops through objects with class of index-card
    for story in stories:
        # Gets text of h2 tag
        title = story.find('h2').get_text()
        # Gets href link of 'a' tag
        story_link = story.find('a').get('href')
        # Gets src from img tag
        game_img = story.find('img').get('src')
        # Gets time updated/posted from class tag
        posted = story.find(class_="entry-meta").get_text()
        # Creates game story object dict
        game_story={'title':title, 'story_link':story_link, 'game_img': game_img, 'posted': posted }
        # Adds game story to game_news list
        game_news.append(game_story)

    context = {
        'game_news': game_news,
    }

    return render(request, 'DadSonGaming/Games/dsg_scrape_game.html', context)
```

## Forms
### Create Game Form

```
class GameForm(ModelForm):
    class Meta:
        model = Game  # Get the Game Model
        fields = '__all__' # Get all fields from Game Model
```

### Create Review Form

```
class ReviewForm(ModelForm):
    class Meta:
        model = Review # Get the Review Model
        fields = '__all__'  # Get all fields from Review Model
```

 