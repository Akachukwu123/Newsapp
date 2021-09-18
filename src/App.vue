<template>
  <div class="container">
    <div id="app">
      <nav class="navbar">
        <h3>World News</h3> 
      </nav>
      <button @click="SavedNews" class="saved_news"> Saved News </button>
      <div class="searchbar">
      <li> 
        <form @submit.prevent="fetchSearchNews">
          <input type="text" placeholder="Search..." v-model="searchword"> 
        </form>
      </li>
      <li> <button class="search" @click="fetchSearchNews">Search</button> </li>
      <li> <button class="refresh" @click="fetchTopNews">×</button> </li>
      </div>
    </div>

    <div class="categories">
      <li> <button id="business" @click="fetchCateNews('business')">Business</button> </li>
      <li> <button id="Entertainment" @click="fetchCateNews('Entertainment')">Entertainment</button> </li>
      <li> <button id="General" @click="fetchCateNews('General')">General</button> </li>
      <li> <button id="Sports" @click="fetchCateNews('Sports')">Sports</button> </li>
      <li> <button id="Technology" @click="fetchCateNews('Technology')">Technology</button> </li>
      <li> <button id="Science" @click="fetchCateNews('Science')">Science</button> </li>
      <li> <button id="Health" @click="fetchCateNews('Health')">Health</button> </li>
    </div>

    <div class="result-list">
      <article v-for ="(article, index) in articles" :key="index">
        <header class="headimage">
          <img v-if="article.urlToImage" :src="article.urlToImage" alt="">
          <img v-else src="assets/logo.png">
        </header>
        <div class="art_title">
        <section v-html="article.title"> </section>
        <p>Source: <b v-html="article.source.name"  @click="fetchNewsSource(article.source.id)"></b> </p>
        </div>
        <footer>
          <button class="now" @click="navTo(article.url)">❯❯ </button> <br> <br>
          <button v-if="saved.includes(article.url)" class="later">Saved</button>
          <button v-else @click="saveToLs(article)" class="later" :id="(article.url)">Save</button>
        </footer>
      </article>

      <button v-on:click="this.fetchNextNews" class="next">More news</button>
    </div>
  </div>
</template>

<script>
export default {
  props: [
    '29eb1ba6c2c74807b370490c56c16975'
  ],
  data:() => {
    return {
      apiUrl:'',
      isBusy: false,
      showloader: false,
      currentPage: 1,
      totalResults: 0,
      maxPerPage: 5,
      searchword: '',
      viewSource : '',
      saved : [],
      articles: [],
      country: 'ng',
      apiKey : '29eb1ba6c2c74807b370490c56c16975'
    }
  },
  methods: {
    navTo(url){
      window.open(url);
    },
    resetData() {
      this.currentPage = 1;
      this.articles = [];
    },
    fetchNewsSource(source){
        this.apiUrl = 'https://newsapi.org/v2/top-headlines?sources=' 
      + source + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
      this.isBusy = true;
      this.viewSource = source;

      this.resetData();
      this.fetchData(this.currentPage);
    },
    fetchSearchNews(){
      if (this.searchword !== '')
      {
        this.apiUrl = 'https://newsapi.org/v2/everything?q=' 
      + this.searchword + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
      this.isBusy = true;

      this.resetData();
      this.fetchData(this.currentPage);
      }
      else{
        this.fetchTopNews();
      }
    },
    fetchTopNews(){
      this.apiUrl = 'https://newsapi.org/v2/top-headlines?country=' 
      + this.country + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
      this.isBusy = true;
      this.searchword = '';
      this.viewSource = '';

      this.resetData();
      this.fetchData(this.currentPage);
    },
    saveToLs(url){
      console.log(localStorage.getItem("myUrl"));
      let items = localStorage.getItem("myUrl");
      let item = JSON.parse(items) == null ? [] : JSON.parse(items);
      console.log(item);
      item.push({url : url});
      localStorage.setItem("myUrl", JSON.stringify(item));
      document.getElementById(url.url).innerText = "Saved";
      this.saved.push(url.url);
    },
     fetchNextNews(){
       if(this.searchword !== ''){
      this.apiUrl = 'https://newsapi.org/v2/everything?q=' 
      + this.searchword + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
     }else if(this.viewSource !== ''){
     this.apiUrl = 'https://newsapi.org/v2/top-headlines?sources=' 
      + this.viewSource + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
     }else{
       this.apiUrl = 'https://newsapi.org/v2/everything?q=' 
      + this.searchword + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
     }
      this.isBusy = true;
      this.currentPage = this.currentPage + 1;
      this.fetchData(this.currentPage);
    },
    fetchCateNews(category){
      document.querySelectorAll(".categories li > button").forEach(element => {
        if(element.id == category){
          element.style.backgroundColor = "#3385ff";
        }else{
          element.style.backgroundColor = "grey"; 
        }
      });
       if(this.searchword == ""){
      this.apiUrl = 'https://newsapi.org/v2/top-headlines?country=' + 
      this.country + '&category=' + category + '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
     }else{
       this.apiUrl = 'https://newsapi.org/v2/everything?q=' 
      + this.searchword + '$country=' + this.country + '&category=' + category +  '&pageSize=' + this.maxPerPage +
      '&apiKey=' + this.apiKey;
     }
      this.isBusy = true;
      this.currentPage = this.currentPage + 1;
      this.fetchData(this.currentPage);
    },
    fetchData(myPage){
      let req = new Request(this.apiUrl + '&page=' + myPage);
      fetch(req)
        .then((resp) => resp.json())
        .then((data) => {
          this.totalResults = data.totalResults;
          console.log(data.articles)
          this.articles = [];
          data.articles.forEach(element => {
            this.articles.push(element);
          });
          this.isBusy = false;
          this.showloader = false;
        })
        .catch((error) => {
          console.log(error);
        })
    },
    SavedNews(){
      let items = localStorage.getItem("myUrl");
      let item = JSON.parse(items) == null ? [] : JSON.parse(items);
      this.articles = [];
      item.forEach(element => {
        console.log(element.url.source);
        this.articles.push(element.url);
      })
    }
  },
  created() {
    this.fetchTopNews();
  }
}
</script>

<style>
  body{
    font-family:calibri;
    margin:0;
  }
  .navbar{
    background:black;
  }
  .slide{
    display: none;
  }

  .saved_news{
    float: right;
    background:#ff3333;
    border-radius: 5px;
    font-size: 12px;
    cursor: pointer;
    color: white;
    padding: 10px 15px;
    border: none;
    margin-top: -20px;
  }
  .navbar h3{
    font-weight:bold;
    margin-left:20px;
    font-size:30px;
    color:white;
  }
  
  .searchbar li{
    list-style:none;
    display:inline-block;
  }
  .refresh{
    margin-left:10px;
    font-size:15px;
    cursor:pointer;
    color:white;
    background:#ff3333;
    border:none;
    border-radius:5px;
    padding:5px 10px;
  }
  .search{
    margin-left:10px;
    cursor:pointer;
    background:#3385ff;
    border:none;
    border-radius:5px;
    padding:10px 10px;
    color:white;
  }
  .searchbar{
    width:320px;
    margin-top:-20px;
    margin-bottom:30px;
    margin-left:auto;
    margin-right: auto;
  }
  .searchbar input{
    border:none;
    background:lightgrey;
    padding:10px 20px;
    border-radius:5px;
    width: calc();
  }
  .container{
    padding:10px;
    height:90vh;
    width: 90%;
    margin-left:auto;
    margin-right:auto;
    border-radius:20px;
  }
  .headimage{
    width: 95%;
  }
  img{
    width: 100%;
    height: 100%;
    object-fit: contain;
    background-color: azure;
  }
  section{
    padding-top: 30px;
  }
  footer{
    padding-top: 30px;
  }
  article{
    display:grid;
    grid-template-rows: 250px;
    grid-template-columns: 400px auto 40px; 
  }
  .next{
    padding:10px 30px;
    font-size: 15px;
    background: #3385ff;
    border: none;
    border-radius: 5px;
    color: white;
    float: right;
    margin-top: 40px;
    margin-bottom: 40px;
    cursor: pointer;
  }
  .now{
    border: none;
    border-radius: 5px;
    background-color: black;
    color: white;
    padding:5px 8px;
    cursor: pointer;
  }
  .later{
    border: none;
    border-radius: 5px;
    background-color: #ff3333;
    color: white;
    padding:5px 8px;
    cursor: pointer;
  }
  .art_title b{
    color: #3385ff;
    cursor: pointer;
  }
  .categories{
    margin-top: -15px;
    margin-bottom: 20px;
    width:60%;
    margin-left: auto;
    margin-right: auto;
  }
  .categories li{
    list-style: none;
    display: inline-block;
  }
  .categories button{
    background:grey;
    color: white;
    border: none;
    border-radius: 5px;
    font-size:13px;
    padding: 10px 20px;
    margin-left: 10px;
    cursor: pointer;
  } 
  .categories button:hover{
    background: grey;
  }






 @media screen and (max-width:800px) {
   article{
    display:grid;
    grid-template-rows: 120px;
    grid-template-columns: 2fr 2.5fr 1fr; 
  }
  .saved_news{
    float: unset;
  }
  section{
    padding-top: 5px;
    padding-left: 10px;
    font-size: 13px;
  }
  footer{
    padding-top: 5px;
    padding-left: 35%;
  }
  .searchbar{
    width: 100%;
    margin-top: 20px;
    margin-bottom:20px;
    margin-left: unset;
    margin-right: unset;
  }
  .navbar{
    border-radius:5px;
  }
  .navbar h3{
    font-size:25px;
    padding: 3px;
  }
  .later{
    font-size: 10px;
  }
  .next{
    padding: 8px 15px;
  }
  
  .categories{
    display: inline-flex;
    overflow-x: auto;
    width:100%;
    padding-bottom: 10px;
  }
  .categories button{
    margin-top: 10px;
    padding: 5px 10px;
  }
  .slide{
    display: unset;
    float: right;
    margin-top:-20px;
    font-size: 9px;
  }

 }
</style>
