<template>
  <div>

    <div class="Title">
      <h1>ForumSquare<div @click="next()">
          <lottie :options="heartOptions" :height="200" :width="300" v-on:animCreated="heartAnimation" />
        </div>
      </h1>
    </div>
    <div @click="Wnext()">
      <lottie :options="WelcomeOptions" :height="300" :width="800" v-on:animCreated="WelcomeAnimation"
        style="margin-top: -60px; margin-bottom: -20px;" />
    </div>

    <div class="block text-center">

      <el-carousel height="700px" indicator-position="outside"
        style="border-radius: 60px; width: 90%; margin-bottom: 50px;">
        <el-carousel-item v-for="articleItem in article" :key="articleItem" style="border-radius: 10px;">

          <h3 class="small justify-center" text="2xl">

            <div class="articlebox">
              <p class="content" style="line-height: 1.2; margin-top: 100px; font-size: 55px;">{{ (articleItem as
                any).content }}</p>
              <div class="flex-container">
                <div class="likescomments">
                  <el-button @click="AddLikes((articleItem as any).id, articleItem)" class="likesbtn"
                    :disabled="(articleItem as any).isLiked" style="font-size: 1.8em; margin-right: 8px;">
                    ❤️{{ (articleItem as any).likes_count }}</el-button>

                  <el-button @click="() => handleEdit((articleItem as any).id, articleItem)" class="likesbtn"
                    style="font-size: 1.8em;">💬 {{ (articleItem as any).comments_count }} </el-button>
                </div>
              </div>
            </div>

          </h3>
        </el-carousel-item>
      </el-carousel>
      <!-- Comment editing box -->
      <CommentDia />
    </div>
  </div>
</template>
    
<script setup lang="ts">

import { reactive, toRefs, watch, onMounted, ref } from 'vue'
import axios from 'axios'
import { useStore } from 'vuex';
import { useRouter } from 'vue-router';
import CommentDia from '../components/CommentDialog.vue'
import Animation1 from '../assets/Animation/HandShaking.json'
import Welcome from '../assets/Animation/Welcome.json'
import Lottie from 'lottie-web';
import lottie from "./Lottie.vue";

// HandingShaking Animation
const heartOptions = {
  animationData: Animation1,
  loop: true,
  autoplay: true,
};

let heartanim = null;
let Direction = -1;

const heartAnimation = (anim) => {
  heartanim = anim;
  if (Direction === -1) {
    (heartanim as any)?.goToAndStop(64, true);
  }
};

const next = () => {
  Direction = -Direction;
  (heartanim as any)?.setDirection(Direction);
  (heartanim as any)?.play();
};

//Welcome
const WelcomeOptions = {
  animationData: Welcome,
  loop: true,
  autoplay: true,
};

let w = null;
let d = -1;

const WelcomeAnimation = (anim) => {
  w = anim;
  if (d === -1) {
    (w as any)?.goToAndStop(64, true);
  }
};

const Wnext = () => {
  d = -d;
  (w as any)?.setDirection(d);
  (w as any)?.play();
};

// Trigger the playback of the animation
onMounted(() => {
  (heartanim as any)?.play();
  (w as any)?.play();
});

let store = useStore()

// Retrieve data for all articles, organize it, and add it to the list
const articlelist = ref({})
const article = ref({})
const GetAllArticle = async () => {
  try {
    const response = await axios.get(`http://localhost:8000/articlelist/`)
    if (response.status == 200) {
      console.log("get all articles success")
      articlelist.value = response.data.data
      Array(articlelist.value).sort((a: any, b: any) => b.likes_count - a.likes_count);
      for (let i = 0; i < 10; i++) {
        article.value[i] = articlelist.value[i]
      }
    }
  } catch (error) {
    console.log(error)
  }
}

onMounted(() => {
  GetAllArticle()
})

// Edit comment 
const handleEdit = (index: number, row: any) => {
  console.log("selected article: ", index, row);
  store.commit("OPEN_DIA", row)
};

const router = useRouter();
const Userdata = reactive({
  id: router.currentRoute.value.query.id,
})

// Add likes
const AddLikes = (index, row) => {
  row.isLiked = true
  try {
    axios.post(`http://localhost:8000/likes/`, {
      article_id: index,
      user_id: Userdata.id,
    })
      .then(function (response) {
        console.log("add likes success!");
        row.likes_count += 1
      })
      .catch(function (error) {
        console.log(error);
      });
  } catch (error) {
    console.log(error)
  }
}

// User data
const user = reactive({
  name: "",
  email: "",
});

const GetUserData = async () => {
  axios.get('http://localhost:8000/login/')
    .then(response => {
      const targetUser = response.data.find(user => user.user_id === Userdata.id);

      if (targetUser) {
        // Handle the found target user data here
        console.log('Target user found');

      } else {
        console.log('Target user not found');
      }

      // data get from backend
      const name = targetUser.name;
      const email = targetUser.email;
      const birthday = targetUser.birthday;
      const account = targetUser.account;

      // Store the data in localStorage
      localStorage.setItem('name', JSON.stringify(name))
      localStorage.setItem('email', JSON.stringify(email))
      localStorage.setItem('birthday', JSON.stringify(birthday))
      localStorage.setItem('account', JSON.stringify(account))

    })
    .catch(error => {
      console.error('Error fetching data:', error);
    });
}

onMounted(() => {
  GetUserData();
})

</script>
  
<style scoped>
.text-center {
  margin-bottom: 10px;
}

.likescomments {
  position: a;
  margin-right: 0px;
  margin-left: 5px;
  bottom: 10px;
  top: 500px;
}

.flex-container {
  display: flex;
  justify-content: flex-end;
  align-items: flex-end;
  margin-bottom: -10px;
}

.content {
  font-size: 30px;
  font-family: 'Nunito', sans-serif;
  font-weight: bold;
}

.likesbtn {
  background-color: transparent;
  padding: 0;
  border: none;
  cursor: pointer;
}

.articlebox {
  box-shadow: 0 2px 4px rgba(0, 0, 0, .12), 0 0 6px rgba(0, 0, 0, .04);
  margin-bottom: 5px;
  margin-top: 5px;
  border-radius: 10px;
  padding: 15px;
  position: relative;
}

#home {
  text-align: center;
  margin-top: 50px;
}

/* Basic styling */
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;

}

/* Center the main title */
h1 {
  color: #333;
  background-color: rgb(91, 186, 126, 0.8);
  padding: 20px;
  margin-bottom: 20px;
  border-radius: 60px;
  width: 70%;
}

.Title {
  text-align: center;
  font-size: 90px;
  font-weight: bold;
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.block {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;

}

.el-container,
.common-layout,
#app,
body,
html {
  height: 100%;

}

.el-col {
  border-radius: 4px;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

.demo-basic {
  text-align: center;
}

.demo-basic .demo-basic--circle,
.demo-basic .demo-basic--square {
  display: flex;
  align-items: center;

  margin-left: 190%;
}

.demo-basic .block:not(:last-child) {
  border-right: 1px solid var(--el-border-color);
}

.demo-basic .block {
  flex: 1;
}

.demo-basic .el-col:not(:last-child) {
  border-right: 1px solid var(--el-border-color);
}


.demonstration {
  color: var(--el-text-color-secondary);
  font-size: 20px;
  position: relative;
}

.el-carousel__item h3 {
  color: #0a0000;
  opacity: 0.75;
  line-height: 520px;
  margin: 0;
  text-align: center;
}

.el-carousel__item:nth-child(2n) {
  background: linear-gradient(to top, #6cc36c, #7dabd8);
}

.el-carousel__item:nth-child(2n + 1) {
  background: linear-gradient(to top, #6cc36c, rgb(198, 115, 198));
}


.demo-type {
  display: flex;
  margin-left: 95%;

}

.demo-type>div {
  flex: 1;
  text-align: center;
}

.demo-type>div:not(:last-child) {
  border-right: 1px solid var(--el-border-color);
}
</style>
    