---
layout: post
title: Membuat Blog menggunakan django dan ReactJS (Lanjutan)
categories: [Web development]
tags: [Django, ReactJS, Django-Rest-Framework, PostgreSQL]
description : Membuat Blog menggunakan django sebagai backend dan ReactJS sebagai frontend dengan database PostgreSQL - Lanjutan
date: 2022-03-04 15:45:00 +0700
---

Setelah [backend](/web%20development/2022/03/04/membuat-blog-menggunakan-django-dan-reactjs), next lanjut buat antarmuka untuk user.

## Frontend

Di dalam folder blog_project buat ReactJS App dan jalankan server.

```python
dany@hp:~/blog_project$ npx create-react-app frontend
dany@hp:~/blog_project$ cd frontend
dany@hp:~/blog_project/frontend$ npm run start
```
Install axio, react-router-dom dan mui/material

```
dany@hp:~/blog_project/frontend$ npm install --save axios react-router-dom @mui/material

```
Pertama-tama kita buat file konfigurasi untuk menyimpan parameter API URL backend yang akan digunakan.

```js
class Config {
    static API_URL =  'http://localhost:8000'
    
}

export default Config;
```

Tambahkan stylesheet font-family Roboto di folder public/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@100;200;300;400;700;900&display=swap" rel="stylesheet">
    
    <title>DanyNotes</title>

    <style>
       *  {
          font-family: 'Roboto', sans-serif;
          margin: 0;
          overflow-x: hidden;
      }

  
    </style>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root">
      
    </div>

  </body>
</html>

```
Kita buat beberapa komponen untuk portfolio ada intro, menu, portfolio, testimonials, topbar, works dan untuk blog terdapat komponen feature post, kategori menu, blog list, blog detail.

So Lanjut Modifikasi App.jsx beserta scss-nya, dan buat Layout.jsx di dalam folder hocs(higher-order Component)

```js

// App.jsx

import React from 'react';
import { BrowserRouter as Router, Switch } from 'react-router-dom';
import Layout from './hocs/Layout';


class App extends React.Component {
    render() {
        return (
            <Router>
                <Switch>
                    <Layout></Layout>
                </Switch>
            </Router>
        );
    }
}

export default App;

```

```js
// hocs/layout.jsx

import React, { useEffect, useState } from 'react';
import Menu from "../components/menu/Menu";
import PersonalWebComponent from '../components/PersonalWebComponent';
import BlogWebComponent from '../components/BlogWebComponent';
import { Link } from 'react-router-dom';
import "../app.scss";
import "../components/topbar/topbar.scss";


const Layout = (props) => {
    const [menuOpen, setMenuOpen] = useState(false);
    const [LinkClicked, setLinkClicked] = useState(false);
    const [HideSection, setHideSection] = useState(false);
    const [HideBlog, setHideBlog] = useState(true);

    // persist state on refresh, Using LocalStorage — Functional Components

    useEffect(() => {
        setHideSection(JSON.parse(window.localStorage.getItem('HideSection')));
        setLinkClicked(JSON.parse(window.localStorage.getItem('LinkClicked')));
        setHideBlog(JSON.parse(window.localStorage.getItem('HideBlog')));
    }, []);

    useEffect(() => {
        window.localStorage.setItem('HideSection', HideSection);
        window.localStorage.setItem('LinkClicked', LinkClicked);
        window.localStorage.setItem('HideBlog', HideBlog);
    }, [HideSection, LinkClicked, HideBlog]);

    return (
        
        <div className="app">
            <div className={"topbar " + (menuOpen && "active") } id="topbar">
                <div className="wrapper">
                    <div className="left">
                        <Link to="/" className="logo" onClick={() => { setMenuOpen(false); setLinkClicked(false); setHideSection(false); setHideBlog(true) }}>DNotes</Link>
                    </div>
                    
                    <div className="right">
                        <div className={"linkmenu " + (LinkClicked && "clicked")}>
                            <Link to="/blog" className="linkto" onClick={() => { setLinkClicked(true); setHideSection(true); setHideBlog(false) } }>Blog</Link>
                        </div>
                        
                        <div className="hamburger" onClick={() => { setMenuOpen(!menuOpen);setLinkClicked(false); setHideSection(false); setHideBlog(true) }}>
                            <span className="line1"></span>
                            <span className="line2"></span>
                            <span className="line3"></span>
                        </div>
                    </div>
                </div>
            </div>

            <Menu menuOpen={menuOpen} setMenuOpen={setMenuOpen}/>
            <PersonalWebComponent HideSection={HideSection}/>
            {props.children}
            <BlogWebComponent HideBlog={HideBlog}/>
        </div>
    );
}

export default Layout;

```

Selanjutnya buat scss dan komponen yang digunakan di layout global.scss, app.scss dan topbar.scss, dan komponen Menu, PersonalWebComponent dan BlogWebComponent

```scss

// app.scss

.app{
    height: 100vh;

    .sections{
        width: 100%;
        height: calc(100vh - 70px);
        background-color: whitesmoke;
        position: relative;
        top: 70px;
        scroll-snap-type: y mandatory;
        scroll-behavior: smooth;
       
        scrollbar-width: none; //for firefox
        &::-webkit-scrollbar{
            display: none;
        }

        > *{
            width: 100vw;
            height: calc(100vh - 70px);
            scroll-snap-align: start;
        }

    }

    .clear {
        
        scrollbar-width: none; //for firefox
        &::-webkit-scrollbar{
            display: none;
        }
    }

    .blog {
        width: 100%;
        padding-top: 70px;
     
    }

  
}
```

```scss

// global.scss

$mainColor: #15023a;

$color-primary: #328da8;
$color-secondary: #47acad;
$color-tertiary: #2fa18a;
$color-quaternary: #87369e;

$color-grey-light: #efefef;
$color-grey-dark: #333;

$color-hover: #edf5f8;
$color-link-hover: #ccc;

$color-paragraph: #4a4a4a;

$color-green: #34eb49;

$color-white: #fff;
$color-black: #000;

// button submit
$bgColor: #fafafa;
$shadow: rgba(lighten($bgColor, 10%),.2);
$accentColor: #1eaacd;
$secondaryColor: #C5C5C5;
$btnWidth: 175px;
$btnHeight: 50px; // Firefox users - don't touch!
$borderRadius: 35px;
$btnBorder: 2px;
$loaderStroke: 3px;
$dribbble: #ea4c89;

// Font
$default-font-size: 1.6rem;

// Grid
$grid-width: 114rem;
$gutter-vertical: 4rem;
$gutter-vertical-small: 2rem;
$gutter-horizontal: 4rem;

$width: 768px;

@mixin mobile {
    @media (max-width: #{$width}){
        @content
    }
}
```

```scss

/* components/topbar/topbar.scss */

@import "../../global.scss";


.topbar {
    width: 100%;
    height: 70px;
    background-color: whitesmoke;
    color: $mainColor;
    position: fixed;
    top: 0;
    z-index: 3;
    transition: all 1s ease;
    overflow-y: hidden;

    .wrapper{
        padding: 10px 30px;
        display: flex;
        align-items: center;
        justify-content: space-between;

        .left{
            display: flex;
            align-items: center;

            .logo{
                font-size: 40px;
                font-weight: 700;
                text-decoration: none;
                color:inherits ;
                margin-right: 40px;
            }

            .itemContainer{
                display: flex;
                align-items: center;
                margin-left: 30px;

                @include mobile {
                    display: none;
                }

                .icon{
                    font-size: 18px;
                    margin-right: 5px;
                }

                span{
                    font-size: 15px;
                    font-weight: 500;
                }
            }

        }

        .right{

            display: flex;
            align-items: center;

            .linkmenu {
                padding-right: 40px;

                .linkto {
                    font-size: 18px;
                    font-weight: 500;
                    text-decoration: none;
                    color: inherits;
                }
                
                &.clicked {
                    cursor: default;
                    pointer-events: none;        
                    text-decoration: none;

                    .linkto {
                        color: grey;
                    }
                }

            }

            .hamburger{
                width: 32px;
                height: 25px;
                display: flex;
                flex-direction: column;
                justify-content: space-between;
                cursor: pointer;

                span{
                    width: 100%;
                    height: 3px;
                    background-color: $mainColor;
                    transform-origin: left;
                    transition: all 2s ease;
                }

            }
        }
    }

    &.active{
        background-color: $mainColor;
        color: white;

        .hamburger{
            span{
                &:first-child{
                    background-color: white;
                    transform: rotate(45deg);
                }
                &:nth-child(2){
                    opacity: 0;
                }
                &:last-child{
                    background-color: white;
                    transform: rotate(-45deg);
                }
            }
            overflow-y: hidden;
        }
    }
}
```

### Component Menu dan scss-nya di folder src/components/menu

```js
// src/components/menu/menu.jsx 

import "./menu.scss";


export default function Menu({ menuOpen, setMenuOpen }) {
    const listmenu = [
        {
            id: 1,
            hastag: "home",
            title: "Home",
            
        },
        {
            id: 2,
            hastag: "portfolio",
            title: "Portfolio",
            
        },
            id: 3,
            hastag: "works",
            title: "Works",
            
        },
        {
            id: 4,
            hastag: "testimonials",
            title: "Testimonials",
            
        },
        {
            id: 6,
            hastag: "contact",
            title: "Contact",
            
        },
    ]
    return (
        <div className={"menu " + (menuOpen && "active")} id="menu">
            <ul>
                {listmenu.map((item) => (
                    <li key={item.id} onClick={()=>setMenuOpen(false)}>  
                        <a href={"/#" + item.hastag}>{item.title}</a>
                    </li>
                ))}
            </ul>
        </div>
    );
}

```

```scss
/* src/components/menu/menu.scss */

@import "../../global.scss";


.menu{
    width: 300px;
    height: 100vh;
    background-color: $mainColor;
    position: fixed;
    top:0;
    right: -300px;
    z-index: 2;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: all 1s ease;

    &.active{
        right: 0;

    }

    ul {
        margin: 0;
        padding: 0;
        list-style: none;
        font-size: 30px;
        font-weight: 300;
        color: white;
        width: 60%;

        li{
            margin-bottom: 25px;
            a{
                font-size: inherit;
                color:inherit;
                text-decoration: none;
            }

            &:hover {
                font-weight: 500;
            }
        }
    }
}
```
### Personal Web Component

```js
// src/components/PersonalWebComponent.jsx

import React from 'react';
import { Route } from 'react-router-dom';
import Home from "./intro/Intro";
import Portfolio from "./portfolio/Portfolio";
import Works from "./works/Works";
import Testimonials from "./testimonials/Testimonials"
import Contacts from "./contact/Contact";
import PropTypes from 'prop-types';


class PersonalWebComponent extends React.Component {
    render() {
        if(this.props.HideSection) {
            return null;
        } else {
            return (
                <div className={"sections"}>
                    <Route exact path="/" component={Home}/>
                    <Route exact path="/" component={Portfolio}/>
                    <Route exact path="/"><Works/></Route>
                    <Route exact path="/"><Testimonials/></Route>
                    <Route exact path="/"><Contacts/></Route>
                </div> 
            );
        }
    }
}

PersonalWebComponent.propTypes = {
    HideSection: PropTypes.bool
}

PersonalWebComponent.defaultProps = {
    HideSection: false
}

export default PersonalWebComponent;
```
Next buat komponen-komponen yang terdapat di Personal Web Component (portfolio) :

#### 1. Intro

```jsx
// src/components/intro/Intro.jsx

import "./intro.scss";
import axios from "axios";
import { init } from 'ityped';
import React, { useState, useEffect, useRef } from "react";
import introimage from "../assets/dany_intro_img.png"
import downarrow from "../assets/down.png"


export default function Intro() {
    const [profile, setProfile] = useState([]);
    const [displayedContent, setDisplayedContent] = useState("");
  
    useEffect(() => {
        const { default: Config } = require("../../utils/config")
        const fetchProfile = async () => {
            try {
                const fullres = await axios.get(Config.API_URL+"/api/blog/profile/");
                setProfile(fullres.data[0].arr_worktitle);
                
            }
            catch (err) {

            }
        }

        fetchProfile();
    }, []);

   

    useEffect(() => {
        setDisplayedContent((displayedContent)=> displayedContent + profile) 
    }, [profile]);

    const textRef = useRef();

    useEffect(() => {
        const strings = [displayedContent]

        init(textRef.current, {
            showCursor: false,
            typeSpeed: 20,
            backDelay: 1500,
            backSpeed: 60,
            strings: [strings],
        });
    }, [displayedContent]);
 
    return (
        <div className="intro" id="home">
            <div className="left">
                <div className="imgContainer">
                    <img src={introimage} alt="" /> 
                </div>
            </div>

            <div className="right">
                <div className="wrapper">
                    <h2>Hi There, I',m</h2>
                    <h1>Dany Christianto</h1>
                    <h3>
                        <span ref={textRef}></span>
                    </h3>
                </div>

                <a href="#portfolio">
                    <img src={downarrow} alt="" />
                </a>
            </div>
        </div>
    );
}
```
```css
/* src/components/intro/intro.scss */

@import "../../global.scss";


.intro {
    background-color: whitesmoke;
    display: flex;

    @include mobile{
        flex-direction: column;
        align-items: center;
    }

    .left{
        flex: 0.5;
        overflow-y: hidden;

        .imgContainer{
            width: 670px;
            height: 670px;
            background-color: whitesmoke;
            border-radius: 50%;
            display: flex;
            align-items: flex-end;
            justify-content: center;
            float: right;

            @include mobile {
                align-items: flex-start;
            }

            img{
                height: 90%;
                margin-bottom: 0%;
                filter: contrast(110%);

                @include mobile{
                    height: 50%;
                }
            }
        }
    }

    .right{
        flex: 0.5;
        display: flex;

        .wrapper{
            width: 100%;
            height: 100%;
            padding-left: 50px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            
            @include mobile {
                padding-left: 0;
                align-items: center;
            }
            h1 {
                font-size: 60px;
                margin: 10px 0;

                @include mobile {
                    font-size: 40px;
                }
            }
            h2 {
                font-size: 35px;
            }
            h3 {
                font-size: 30px;
            

                @include mobile {
                    font-size: 20px;
                }

                span{
                    font-size: inherit;
                    color: crimson;
                }

                .ityped-cursor {
                    animation: blink 1s infinite;
                }

                @keyframes blink {
                    50%{
                        opacity: 1;
                    }
                    100%{
                        opacity: 0;
                    }
                     
                }
            }

            .simple-cloud .tag-cloud-tag {
                flex-direction: column;
                justify-content: center;
            }
        }

        a {
            position: absolute;
            bottom: 10px;
            left: 60%;
      
            img {
              width: 30px;
              animation: arrowBlink 2s infinite;
            }
      
            @keyframes arrowBlink {
              100% {
                opacity: 0;
              }
            }
        }

        .typed {
            display: inline;
        }

    }
}
```

#### 2. Portfolio

Didalam komponen portfolio ini kita ambil data list kategori portfolio dan data portfolio berdasarkan id kategori yang dipilih menggunakan django rest framework.

```js
// src/components/portfolio/Portfolio.jsx

import PortfolioList from "../portfoliolist/PortfolioList"
import "./portfolio.scss"
import React, { useEffect, useState } from "react";
import axios from "axios";


export default function Portofolio() {
    const [selected, setSelected] = useState(1);
    const [data, setData] = useState([]);
    const [pfcates, setPfcates] = useState([]);
    
    useEffect(() => {
     
        // load data list portfolio dari DRF
        const fetchListCates = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.get(Config.API_URL +"/api/blog/portofolio_category/");
                setPfcates(res.data);
            }
            catch (err) {

            }
        }

        fetchListCates();
    }, []);


    useEffect(() => {
        const porto_cat_id = selected;

        const config = {
            headers: {
                'Content-Type': 'application/json'
            }
        };

        // load data portfolio berdasarkan list kategori yang dipilih
        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.post(Config.API_URL + `/api/blog/portofolios/`, { porto_cat_id }, config );
                setData(res.data);
            }
            catch (err) {

            }
        };

        fetchData();
    }, [selected]);

    return (
        <div className="portofolio" id="portfolio">
            <h1>Portfolio</h1>
            <ul>
                {pfcates.map((item) => (
                    <PortfolioList
                        key={item.id}
                        title={item.port_cat_name}
                        active={selected === item.id}
                        setSelected={setSelected}
                        id={item.id}
                    />
                ))}
            </ul>
            <div className="container">
                {data.map((d) => ( 
                    <div key={d.id} className="item">
                        <h3>{d.porto_title}</h3>
                        <img
                            src={d.porto_thumbnail}
                            alt=""
                        />
                    </div>
                ))}
            </div>
        </div>
    );
}
```
```css
/* src/components/portfolio/portfolio.scss */

@import "../../global.scss";


.portofolio { 
    background-color: whitesmoke;
    display: flex;
    flex-direction: column;
    align-items: center;

    h1{
        font-size: 50px;

        @include mobile {
            font-size: 20px;
        }
    }

    ul{
        margin:10px;
        padding: 0;
        list-style: none;
        display: flex;

        @include mobile {
            margin: 10px 0;
            flex-wrap: wrap;
            justify-content: center;
        }

        li {
            font-size: 12px;
            margin-right: 5px;
            padding: 0.7;
            border-radius: 10px;
            cursor: pointer;

            &.active {
                background-color: $mainColor;
                color: white;
            }
        }
     
    }

    .container{
        width: 70%;
        display: flex;
        align-items: center;
        justify-content: center;
        flex-wrap: wrap;

        @include mobile {
            width: 100%;
        }

        .item{
            width: 620px;
            height: 400px;
            border-radius: 20px;
            border: 1px solid rgb(240,239,239);
            margin: 10px 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            position: relative;
            transition: all .5s ease;
            cursor: pointer;

            h3{
                position: absolute;
                font-size: 12px;
            }

            img{
                width: 100%;
                height: 100%;
                object-fit: cover;
                z-index: 1;

            }

            &:hover{
                background-color: $mainColor;
                img {
                    opacity: 0.1;
                    z-index: 0;
                }

            }
        }
    }
}

```

```js
// src/components/portfoliolist/Portfoliolist.jsx

import "./portfoliolist.scss"


export default function portfoliolist({ id, title, active, setSelected }) {
    return (
        <li key={id}
            className={active ? "portfoliolist active" : "portfoliolist"} 
            onClick={()=>setSelected(id)}
        >
            {title}
        </li>
    )
}
```

```css
/* src/components/portfoliolist/portfolios.scss */

@import "../../global.scss";


.portfoliolist {
    font-size: 14px;
    margin-right: 50px;
    padding: 7px;
    border-radius: 10px;
    cursor: pointer;

    @include mobile {
        margin-right: 20px;
    }

    &.active{
        background-color: $mainColor;
        color: whitesmoke;
    }
}
```

#### 3. Works

Komponen works ini untuk menampilkan daftar project yang pernah dikerjakan.

```js
// src/components/works/Works.jsx

import { useState, useEffect } from 'react';
import "./works.scss";
import axios from "axios";


export default function Works() {
    const [currentSlide, setCurrentSlide] = useState(0)
    const [data, setData] = useState([])

    const regex = /(<([^>]+)>)/ig;
    
    // function untuk menghilangkan html tag dari data text
    const RemoveHtmlTag = (word) => {
        if (word)
            return word.replace(regex, '');
        return '';
    };


    useEffect(() => {
        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const fullrest = await axios.get(Config.API_URL + "/api/blog/jobs/");
                setData(fullrest.data);
            }
            catch (err) {

            }
        }
        fetchData();

    }, [])

    const handleClick = (way)=>{
        way === "left" 
        ? setCurrentSlide(currentSlide > 0 ? currentSlide - 1 : 2)
        : setCurrentSlide(currentSlide < data.length - 1 ? currentSlide + 1 : 0);
    }

    return (
        <div className="works" id="works">
          
            <div 
                className="slider" 
                style={ transform: `translateX(-${currentSlide * 100}vw)`}
            >
                {data.map((d) => (
                    <div className="container" key={d.id}>
                        <div className="item">
                            <div className="left">
                                <div className="leftContainer">
                                    <div className="imgContainer">
                                        <img src={d.job_icon} alt="" />
                                    </div>
                                    <h2>{d.job_name}</h2>
                                    
                                    <p>{RemoveHtmlTag(d.job_desc)}</p>
                                   
                                    <span>project</span>
                                </div>
                            </div>
                            <div className="right">
                                <img src={d.jobs_thumbnail} alt="" />
                            </div>
                            
                        </div>

                    </div>
                ))}
            </div>
            <img 
                src="assets/arrow.png" 
                className="arrow left" 
                alt="" 
                onClick={()=>handleClick("left")}
            />
            <img 
                src="assets/arrow.png" 
                className="arrow right" 
                alt=""
                onClick={()=>handleClick("right")}
            />
        </div>
    );
}
```

```css
/* src/components/works/works.scss */

@import "../../global.scss";


.works {
    background-color: crimson;
    display: flex;
    align-items: center;
    justify-content: center;
  
    .arrow {
        height: 50px;
        position: absolute;

        @include mobile {
            display: none;
        }

        &.left {
            left: 100px;
            transform: rotateY(180deg);
        }

        &.right {
            right: 100px;
        }
    }

    .slider {
        height: 350px;
        display: flex;
        position: absolute;
        left: 0;
        transition: all 1s ease;

        @include mobile {
            height: 100vh;
            flex-direction: column;
            justify-content: center;
        }

        .container {
            width: 100vw;
            display: flex;
            align-items: center;
            justify-content: center;
            
            .item {
                width: 700px;
                height: 100%;
                background-color: white;
                border-radius: 20px;
                display: flex;
                align-items: center;
                justify-content: center;

                @include mobile {
                    width: 80%;
                    height: 150px;
                    margin: 15px 0;
                }
                

                .left{
                    flex: 4;
                    height: 80%;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    
                    .leftContainer{
                        width: 90%;
                        height: 90%;
                        display: flex;
                        flex-direction: column;
                        justify-content: space-between;
                        
                        .imgContainer{
                            width: 40px;
                            height: 40px;
                            border-radius: 40px;
                            background-color: rgb(245, 179, 155);
                            display: flex;
                            align-items: center;
                            justify-content: center;

                            @include mobile {
                                width: 20px;
                                height: 20px;
                            }

                            img {
                                width: 25px;

                                @include mobile {
                                    font-size: 15px;
                                }
                            }
                        }

                        h2{
                            font-size: 20px;
                            padding-top: 10%;
                            overflow-y: hidden;

                            @include mobile {
                                font-size: 13px;
                            }
                        }

                        p{
                            font-size: 13px;
                            padding-top: 5%;
                            overflow-y: hidden;

                            @include mobile {
                                display: none;
                            }
                        }

                        span {
                            padding-top: 5%;
                            font-size: 12;
                            font-weight: 600;
                            text-decoration: underline;
                            cursor: pointer;
                            overflow-y: hidden;
                        }
                    }
                }

                .right {
                    flex: 8;
                    height: 100%;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    overflow: hidden;

                    img {
                        width: 400px;
                        transform: rotate(-10deg);
                       
                    }
                }
            }
        }
    }
}
```

#### 4. Testimonials

Komponen testimonials ini menampilkan data testimonial yang di input dari admin panel

```js
// src/components/testimonials/testimonials.jsx

import './testimonial.scss'
import axios from 'axios';
import React, { useState, useEffect } from 'react';


export default function Testimonials() {
    const regex = /(<([^>]+)>)/ig;
    const RemoveHtmlTag = (word) => {
        if (word)
            return word.replace(regex, '');
        return '';
    };
    const [testimony, setTestimony] = useState([]);

    useEffect(() => {
        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const resp = await axios.get(Config.API_URL+"/api/blog/testimonial/");
                setTestimony(resp.data);
            }
            catch (err) {

            }
        }
        fetchData();
    }, []);


    return (
        <div className="testimonials" id="testimonials">
            <h1>Testimonials</h1>
            <div className="container">
                {testimony.map((d) => ( 
                    <div key={d.id} className={d.testimonial_featured ? "card featured" : "card"}>
                        <div className="top">
                            <img src="assets/right-arrow.png" className="left" alt=""/>
                            <img className="user"
                                src={d.testimonial_img}
                                alt="" 
                            />
                            <img className="right" src={d.testimonial_icon} alt="" />
                        </div>
                        <div className="center">
                            {RemoveHtmlTag(d.testimonial_desc)}
                        </div>
                        <div className="bottom">
                            <h3>{d.testimonial_name}</h3>
                            <h3>{d.testimonial_title}</h3>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    )
}
```

```css
/* src/components/testimonials/testimonials.scss */

@import "../../global.scss";

.testimonials{
    background-color: whitesmoke ;
    display: flex;
    flex-direction: column;
    align-items: center;

    @include mobile{
        justify-content: space-around;
    }   
    
    h1{
        font-size: 50px;
        overflow: hidden;
        margin-bottom: 20px;

        @include mobile {
            font-size: 20px;
        }
    }

    .container {
        width: 100%;
        height: 80%;
        display: flex;
        align-items: center;
        justify-content: center;

        @include mobile {
            flex-direction: column;
            height: 100%;
        }

        .card {
            width: 250px;
            height: 70%;
            border-radius: 10px;
            box-shadow: 0px 0px 15px -8px black;
            display:  flex;
            flex-direction: column;
            justify-content: space-around;
            padding: 20px;
            transition: all 1s ease;

            @include mobile {
                height: 180px;
                margin: 10px 0;
            }

            &.featured {
                width: 300px;
                height: 75%;
                margin: 0 30px;

                @include mobile {
                    width: 250px;
                    height: 180px;
                    margin: 1px;
                }

            }

            &:hover {
                transform: scale(1.1);
            }

            .top {
                display: flex;
                align-items: center;
                justify-content: center;
    
                img {
                    &.left,
                    &.right {
                        height: 25px;
                    }

                    &.user {
                        height: 60px;
                        width: 60px;
                        border-radius: 50%;
                        object-fit: cover;
                        margin: 0 30px;

                        @include mobile{
                            width: 30px;
                            height: 30px;
                        }
                    
                    }
                }
            }
    
            .center {
                padding: 10px;
                border-radius: 10px;
                background-color: rgb(250, 244, 245);

                @include mobile{
                    font-size: 12px;
                    padding: 5px;
                }
        
            }
              
            .bottom {
                display: flex;
                align-items: center;
                flex-direction: column;
                justify-content: center;
        
                h3 {
                  margin-bottom: 5px;


                  @include mobile{
                    font-size: 14px;
                  }
                }
            
                h4{
                    color: rgb(121, 115, 115);

                    @include mobile{
                        font-size: 13px;
                    }
                }
            }
        }
    }
}
```
#### 5. Contact Form

Seperti contact form pada umumnya, contact form ini berfungsi sebagai user interface untuk pengguna mengirimkan pesan ke admin web. contact form ini dilengkapi dengan fitur send email ke email admin sebagai notifikasi adanya pesan masuk dari web.

```js
// src/components/contact/Contact.jsx

import "./contact.scss";
import React, { useState, useEffect } from "react";
import axios from 'axios';
import { Oval } from 'react-loader-spinner';
import { FormGroup } from '@mui/material';
import { Input } from '@mui/material';
import TextField from '@mui/material/TextField'
import toast, { Toaster } from 'react-hot-toast';
import mailcontact_img from '../assets/mailcontact.png'


export default function Contact () {
    useEffect(() => {
        window.scrollTo(0, 0);
    }, []);
   
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        message: ''
    });

    const { name, email, message } = formData;
    const [loading, setLoading] = useState(false);
    const onChange = e => setFormData(
        { ...formData, [e.target.name]: e.target.value }
    );
    const onSubmit = async (e) => {
        const { default: Config } = require("../../utils/config")        
        e.preventDefault();

        try {
            const config = {
                headers: {
                    'Content-Type': 'application/json'
                }
            };
                
            setLoading(true);
            axios.post(Config.API_URL + `/api/blog/contact/`, {name, email, message }, config)
            .then(res => {

                if (res.status === 200) {
                    setFormData({
                        name: '',
                        email: '',
                        message: ''
                    });

                    toast.success('Thank You, your message was sent', {
                    
                        // Styling
                        className: 'Toast-Message',
                        icon: '',
                    })
                }
                setLoading(false);
                window.scrollTo(0, 0);
            })
            .catch(err => {
                toast.success('Error sending message', {
                    // Styling
                    className: 'Toast-Message',
                        icon: '',
                })
              
                setLoading(false);
                window.scrollTo(0, 0);
            })

        } 
        catch (err) {
            console.log(err);
        }
    };

    return (
        <div className="contact" id="contact">
            <div className="left">
                <div className="Container">
                    <img src={mailcontact_img} alt="" />
                </div>
            </div>
            <div className="right">
                <h2>Contact</h2><br/>
                <form className="contact__form" id="contact-form" onSubmit={e => onSubmit(e)}>
                    <FormGroup sx={{width: '40ch',}}> 
                        <Input 
                            className='contact__form__input' 
                            name='name' 
                            type='text' 
                            placeholder='Full Name' 
                            onChange={e => onChange(e)} 
                            value={name} 
                            required 
                        />
                        <br/>
                        <Input 
                            className='contact__form__input' 
                            name='email' 
                            type='email' 
                            placeholder='example@mail.com' 
                            onChange={e => onChange(e)} 
                            value={email} 
                            required 
                        />
                        <br/>
                        <TextField
                            className='contact__form__textarea'
                            name='message'
                            multiline
                            rows={10}
                            placeholder='Message'
                            onChange={e => onChange(e)} 
                            value={message} 
                        />
                    </FormGroup>
                    <br/>
                 
                    <button 
                        className='contact__form__button' 
                        htmltype='submit'
                        disabled={loading}
                        
                    >
                        {loading ? 
                            <div className="loader-message">
                                <Oval 
                                    color={'#424242'}
                                    height={20} 
                                    width={20}
                                /> Sending...
                            </div> 
                            : 'Send'
                        }
                    </button>
                </form>
            </div>

            <Toaster
                position="top-right" 
                reverseOrder={false} 
            />
        </div>
    );
};
```

```css
/* src/components/contact/contact.scss */

@import "../../global.scss";

.contact {
    background-color: whitesmoke;
    display: flex;

    @include mobile {
        flex-direction: column;
    }

    .left{
        flex:1;
        overflow: hidden;
        flex-direction: column;

        .Container{
            width: 700px;
            height: 650px;
            border-radius: 30%;
            display: flex;
            align-items: flex-end;
            justify-content: center;
            float: right;
                    
            img {
                height: 100%;
            }
        }
    }

    .right {
        flex: 1;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    
        h2 {
            font-size: 30px;
            overflow: hidden;
            height: 15%;
        }

        .contact__form__button {
            outline: none;
            border: none;
            cursor: default;
            transition: box-shadow 300ms ease;
            letter-spacing: 2px;
            box-shadow: 0px 0px $btnHeight/2 20px $shadow;
            user-select: none;
  
            position: relative;
            box-sizing: border-box;
            width: $btnWidth;
            height: $btnHeight;
            border-radius: $borderRadius;
            border: $btnBorder solid;
            border-color: $accentColor;
            background: none;
            color: $accentColor;
            transition: all 300ms ease-out;
            
            &:hover {
                cursor: pointer;
                font-size: 1.05em;
                border-color: transparent;
                background: $accentColor;
                color: $bgColor;
            }

            .loader-message {
                font-size: 0.9em;
                display: flex;
                align-items: center;
                justify-content: center;
                
            } 
        }
    }
}
```
### Blog Web Component

Komponen Blog web ini berfungsi sebagai halaman web blog.

```js
// src/components/blog/blog.jsx

import React, { useState, useEffect } from 'react';
import { Route } from 'react-router-dom';
import axios from 'axios';
import PropTypes from 'prop-types';

import { createTheme, ThemeProvider } from '@mui/material/styles';
import CssBaseline from '@mui/material/CssBaseline';
import Container from '@mui/material/Container';

import Blog from './blog/Blog'
import Category from './blog/Category';
import BlogDetail from './blog/BlogDetail';
import Header from './blog/Header';
import Footer from './blog/Footer';


const BlogWebComponent = (props) => {
    const [sectionBlog, setSectionBlog] = useState([]);

    useEffect(() => {
        const fetchSection = async () => {
            const { default: Config } = require("../utils/config")
            try {
                const res = await axios.get(Config.API_URL + "/api/blog/list_category/");
                setSectionBlog(res.data)
            }
            catch (err) {

            }
        }
        fetchSection();
    }, []);
    
    const theme = createTheme();

    if(props.HideBlog) {
        return null;
    }
    else {
        return (
            <div className="blog">
                <ThemeProvider theme={theme}>
                    <CssBaseline />
                    <Container maxWidth="lg" sx={{ overflow: 'hidden'}}>
                        <Header title="Blog" sections={sectionBlog}/>
                            
                        <Route exact path='/blog' component={Blog}/>
                        <Route exact path='/blog/category/:cat_title_id' component={Category}/>
                        <Route exact path='/blog/blogdetail/:id' component={BlogDetail}/>
                    </Container>
                    <Footer
                        title="Footer"
                        description="Something here to give the footer a purpose!"
                    />
                </ThemeProvider>          
            </div>
        );
    }
 
}

BlogWebComponent.propTypes = {
    HideBlog: PropTypes.bool
}

BlogWebComponent.defaultProps = {
    HideBlog: true
}

export default BlogWebComponent;
```

Komponen Web Blog terdiri dari beberapa komponen berikut : 

#### 1. Header
Komponen header ini menampilkan menu katergori.

```js
// src/components/blog/Header.jsx

import React from 'react';
import PropTypes from 'prop-types';
import Toolbar from '@mui/material/Toolbar';
import Box from '@mui/material/Box';
import Link from '@mui/material/Link';

import { useHistory } from "react-router-dom";
import IconButton from '@mui/material/IconButton';
import SvgIcon from '@mui/material/SvgIcon';
import "./header.scss";


function HomeIcon(props) {
  return (
    <SvgIcon {...props}>
      <path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z" />
    </SvgIcon>
  );
}


const Header = (props) => {
    const { sections } = props;
    
    const history = useHistory();
    const handleClickBlogHome = () => {
        history.push('/blog')
    };

    const CapitalizeFirstLetter = (word) => {
        if (word) 
            return word.charAt(0).toUpperCase() + word.slice(1);
        return '';
    }; 
  
    return (
        <React.Fragment>
            <Box sx={{ flexGrow: 1 }}>
                <Toolbar
                    component="nav"
                    variant="dense"
                    sx = {{
                        justifyContent: 'space-between',
                        overflowX: 'auto'
                    }}
                >
                    <IconButton
                        size="small"
                        color="inherit"
                        aria-label="menu"
                        onClick={handleClickBlogHome}
                    >
                        <HomeIcon />
                    </IconButton>
                    
                        {sections.map((section) => (
                            <Link
                                underline="none" 
                                color="inherit"
                                noWrap
                                key={section.id}
                                variant="body2"
                                href={`/blog/category/${section.id}`}
                                sx={{ p: 1, flexShrink: 0}}
                                
                            >
                                {CapitalizeFirstLetter(section.cat_title)}
                            </Link>

                            
                        ))}
                    
                </Toolbar>
            </Box>
        </React.Fragment>
    );
}
  
Header.propTypes = {
    sections: PropTypes.arrayOf(
      PropTypes.shape({
        cat_title: PropTypes.string.isRequired,
        id: PropTypes.number.isRequired,
      }),
    ).isRequired,
    title: PropTypes.string.isRequired,
};
  
export default Header;
```
```scss
/* src/components/blog/header.scss */

@import "../../global.scss";


.MenuLink {
    color: inherits;
    &.clicked {
        cursor: default;
        pointer-events: none;        
        text-decoration: none;
        color: gey;
    }
}
```

#### 2. Blog
Komponen Blog ini berfungsi untuk mengambil data blog post kemudian di tampilkan menggunakan komponen Main.

```js
// src/components/blog/Blog.jsx

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import Main from './Main';


export default function Blog() {
    const [blogs, setBlogs] = useState([]);

    useEffect(() => {
        const fetchBlogs = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.get(Config.API_URL + "/api/blog/");
                setBlogs(res.data);
            }
            catch (err) {

            }
        }
        fetchBlogs();
    }, []);

    return (
        <Main posts={blogs}/>
    );
}
```

```js
// src/components/blog/Main.jsx

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import Grid from '@mui/material/Grid';
import FeaturedPost from './FeaturedPost';
import MainFeaturedPost from './MainFeaturedPost';

export default function Main(props) {
    const { posts } = props;
    const [featuredBlog, setFeaturedBlog] = useState([]);

    const capitalizeFirstLetter = (word) => {
        if (word)
            return word.charAt(0).toUpperCase() + word.slice(1);
        return '';
    };

    useEffect(() => {
        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.get(Config.API_URL + `/api/blog/featured`);
                setFeaturedBlog(res.data[0]);
            }
            catch (err) {

            }
        }
        fetchData();
    }, []);

    return (
        <main>
            <MainFeaturedPost post={featuredBlog}/>
            <Grid container spacing={3}>
                {posts.map(post => (
                    <FeaturedPost key={capitalizeFirstLetter(post.title)} post={post} />
                ))}
            </Grid>

        </main>
        
      
    );
}
```
Didalam komponen Main terdapat komponen MainFeaturedPost yang berfungsi menampilkan ***post featured***

```js
// src/components/blog/MainFeaturedPost.jsx

import * as React from 'react';
import Paper from '@mui/material/Paper';
import Typography from '@mui/material/Typography';
import Grid from '@mui/material/Grid';
import Link from '@mui/material/Link';
import Box from '@mui/material/Box';


function MainFeaturedPost(props) {
    const { post } = props;

    return (
        <Paper
            sx={
            position: 'relative',
            backgroundColor: 'grey.800',
            color: '#fff',
            mb: 4,
            backgroundSize: 'cover',
            backgroundRepeat: 'no-repeat',
            backgroundPosition: 'center',
            backgroundImage: `url(${{post.thumbnail}})`,
            }
        >   
            {/* Increase the priority of the hero background image */}
            {<img style={{ display: 'none' }} src={post.thumbnail} alt={post.imageText} />}
            
            <Box
                sx={
                    position: 'absolute',
                    top: 0,
                    bottom: 0,
                    right: 0,
                    left: 0,
                    backgroundColor: 'rgba(0,0,0,.3)',
                }
            />
            <Grid container>
                <Grid item md={6}>
                    <Box
                        sx={
                            position: 'relative',
                            p: { xs: 3, md: 6},
                            pr: { md: 0},
                        }
                    >
                        <Typography component="h1" variant="h3" color="inherit" overflow="hidden" gutterBottom>
                            {post.title}
                        </Typography>
                        <Typography variant="h5" color="inherit" paragraph>
                            {post.excerpt}
                        </Typography>
                        <Link variant="subtitle1" href={`/blog/blogdetail/${post.slug}`}>
                            Continue reading...
                        </Link>
                    </Box>
                </Grid>
            </Grid>
        </Paper>
    );
}

export default MainFeaturedPost;
```

#### 3. Blog Detail
Komponent detail berfungsi menampilkan detail post artikel.

```js
// src/components/blog/BlogDetail.jsx

import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import dateFormat from 'dateformat';
import axios from 'axios';
import Box from '@mui/material/Box';
import Typography from '@mui/material/Typography';


const BlogDetail = (props) => {
    const [PostDetail, setPostDetail] = useState({});

    useEffect(() => {
        const slug = props.match.params.id;
        
        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.get(Config.API_URL + `/api/blog/${slug}`);
                setPostDetail(res.data);
            }
            catch (err) {
                
            }
            
        };
        fetchData();
    }, [props.match.params.id]);

    const createBlog = () => {
        return {__html: PostDetail.content}
    };

    const CapitalizeFirstLetter = (word) => {
        if (word)
            return word.charAt(0).toUpperCase() + word.slice(1);
        return '';
    };

    return (
        <Box mt={4}>
            <Typography variant="h4">
                {CapitalizeFirstLetter(PostDetail.title)}
            </Typography>
            <Typography variant="subtitle1">
                {dateFormat(PostDetail.date_created, "mmm dS, yyyy")}
            </Typography>
            <Typography mt={3} paragraph gutterBottom>
                <span align='justify' dangerouslySetInnerHTML={createBlog()}/>
            </Typography>

            <Link to='/blog' color="primary" underline="none">Back to Blog</Link>
        </Box>
    );

};

export default BlogDetail;
```

#### 4. Category
Komponen ini menampilkan artikel (post) berdasarkan kategori yang di pilih di menu kategori

```js
// src/components/blog/Category.jsx

import React, { useState, useEffect} from 'react'

import Card from '@mui/material/Card';
import CardContent from '@mui/material/CardContent';
import CardMedia from '@mui/material/CardMedia';
import { CardActionArea, CardActions } from '@mui/material';
import Typography from '@mui/material/Typography';
import Link from '@mui/material/Link';
import Grid from '@mui/material/Grid';

import axios from 'axios';


const Category = (props) => {
    const [blogs, setBlogs] = useState([]);
    
    useEffect(() => {
        const cat_title_id = props.match.params.cat_title_id;
    
        const cfg = {
            headers: {
                'Content-Type': 'application/json'
            }
        };

        const fetchData = async () => {
            const { default: Config } = require("../../utils/config")
            try {
                const res = await axios.post(Config.API_URL + "/api/blog/category", { cat_title_id }, cfg);
                setBlogs(res.data);

            }
            catch (err) {

            }
        };
        fetchData();
    }, [props.match.params.cat_title_id]);

    const CapitalizeFirstLetter = (word) => {
        if (word)
            return word.charAt(0).toUpperCase() + word.slice(1);
        return '';
    };

    return (
        <Grid container spacing={3}>
            {blogs.map(post => (
                <Grid item xs={12} md={6} key={post.id}>
                    <CardActionArea>
                        <Card sx={{ display: 'flex' }}>
                            <CardContent sx={{ flex:1 }} >
                                <Typography variant="subtitle1" color="text.secondary">
                                    <Link href={`/blog/category/${post.cat_title_id}`} color="primary" underline="hover">{post.cat_title}</Link>
                                </Typography>
                                <Typography component="h2" variant="h5">
                                    {CapitalizeFirstLetter(post.title)}
                                </Typography>
                                <Typography variant="subtitle1" color="text.secondary">
                                    {post.month} {post.day}
                                </Typography>
                                <Typography variant="subtitle1" paragraph>
                                    {post.excerpt}
                                </Typography>
                                        
                                <CardActions>
                                    <Link href={`/blog/blogdetail/${post.slug}`} color="primary" underline="none">Continue reading...</Link>
                                </CardActions>
                            </CardContent>   
                            <CardMedia
                                component="img"
                                sx={ width: 200, heigth: 250, display: { xs: 'none', sm: 'block' } }
                                image= {post.thumbnail} 
                                alt={post.slug}
                            />
                        </Card>
                    </CardActionArea>
                </Grid>
            ))}
        </Grid>
    )
};

export default Category;
```

Setelah semua komponen selesai dibuat, selanjutnya lakukan build ***npm run build*** dari folder frontend dan copy folder hasil build ke dalam directory backend/nama aplikasi. setelah itu jalankan django dan yea.. aplikasi web portfolio dan blog dengan reactjs sudah bisa tampil di django service.

