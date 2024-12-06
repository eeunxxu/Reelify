<template>
  <div v-if="userData && userData.username" class="container my-5">
    <h1 class="text-center mt-3">{{ userData.username }}의 영화 취향</h1>
    <div
      class="user-profile d-flex flex-row justify-content-between align-items-center"
    >
      <div class="position-relative">
        <!-- 프로필 이미지 -->
        <img :src="profileImgUrl" class="user-img" alt="profile_img" />

        <!-- 파일 업로드 버튼 -->
        <div
          v-if="userData.username === store.userName"
          class="position-absolute bottom-0 end-0 mb-2 me-2"
        >
          <input
            type="file"
            ref="fileInput"
            class="d-none"
            @change="uploadProfileImage"
          />
          <button @click="triggerFileInput" class="badge rounded-pill">
            편집
          </button>
        </div>
      </div>
      <div class="profile-text text-center">
        <h3 class="mb-4 username">{{ userData.username }}</h3>
        <div class="follow-text d-flex flex-row mx-2">
          <p @click="goToUserFollower(userData.username)">
            팔로우&ensp;<strong>{{ userData.followers_count }}</strong
            >&ensp;
          </p>
          <p @click="goToUserFollowing(userData.username)">
            &ensp;팔로잉&ensp;<strong>{{ userData.followings_count }}</strong>
          </p>
        </div>
        <div class="userLikeGenre">
          # {{ userLikeGenre[0].name }} # {{ userLikeGenre[1].name }} #
          {{ userLikeGenre[2].name }}
        </div>
      </div>
      <div
        v-if="
          store.isLogin === true &&
          store.userName !== userData.username &&
          isFollow != null &&
          isFollow === false
        "
      >
        <button class="follow-btn" @click="followUser(userData.username)">
          팔로우
        </button>
      </div>
      <div
        v-else-if="
          store.isLogin === true &&
          store.userName !== userData.username &&
          isFollow != null &&
          isFollow === true
        "
      >
        <button class="unFollow-btn" @click="unFollowUser(userData.username)">
          팔로잉
        </button>
      </div>
      <div
        v-else-if="
          store.isLogin === true && store.userName === userData.username
        "
      >
        <button class="update-btn" @click="goToUpdate(userData.username)">
          내 정보 수정
        </button>
      </div>
    </div>

    <!-- 영화, 작성 - 추천 리뷰 보여주기 -->
    <div class="">
      <hr />
      <div class="d-flex flex-row justify-content-evenly mt-4">
        <div class="text-center">
          <p>추천한 영화 수</p>
          <span>{{ likeMovieCnt }}</span>
        </div>
        <div class="text-center">
          <p>작성한 리뷰 수</p>
          <span
            class="review-cnt"
            v-if="writeReviewCnt > 3"
            @click="goToUserReviewList(userData.username)"
          >
            {{ writeReviewCnt }}
          </span>
          <span v-else>{{ writeReviewCnt }}</span>
        </div>
        <div class="text-center">
          <p>추천한 리뷰 수</p>
          <span
            class="likeReview-cnt"
            v-if="likeReviewCnt > 3"
            @click="goToUserLikeReview(userData.username)"
            >{{ likeReviewCnt }}</span
          >
          <span v-else>{{ likeReviewCnt }}</span>
        </div>
      </div>
      <hr />
    </div>

    <UserRatingGraph />

    <RouterView />
    <!-- 좋아요 한 영화 목록 보여주기 -->
    <div>
      <h2>{{ userData.username }}님이 추천하는 영화</h2>
      <UserLikeMovie :likemovie="likeMovie" />
      <hr />
    </div>
    <!-- 작성한 리뷰 보여주기 -->
    <div>
      <h2>{{ userData.username }}님이 작성한 리뷰</h2>
      <div
        class="d-flex justify-content-end"
        v-if="
          userData &&
          userData.written_reviews &&
          userData.written_reviews.length > 3
        "
      >
        <button
          class="click-btn"
          @click="goToUserReviewList(userData.username)"
        >
          전체 리뷰 / 댓글 보기 + {{ writeReviewCnt }}
        </button>
      </div>
      <div class="mt-4 d-flex flex-column align-items-center">
        <UserReviewCard
          class="mb-4"
          v-for="review in limitedReviews"
          :key="review.id"
          :review="review"
        />
      </div>
    </div>
    <hr />

    <!-- 추천한 리뷰 보여주기 -->
    <div>
      <h2>{{ userData.username }}님이 추천한 리뷰</h2>
      <div
        class="d-flex justify-content-end"
        v-if="
          userData &&
          userData.liked_reviews &&
          userData.liked_reviews.length > 3
        "
      >
        <button
          class="click-btn"
          @click="goToUserLikeReview(userData.username)"
        >
          전체 리뷰 / 댓글 보기 +{{ likeReviewCnt }}
        </button>
      </div>
      <div
        class="mt-4 d-flex flex-column align-items-center"
        v-if="
          userData &&
          userData.liked_reviews &&
          userData.liked_reviews.length > 0
        "
      >
        <UserLikeReview
          class="mb-4"
          v-for="review in limitedLikeReviews"
          :key="review.id"
          :review="review"
        />
      </div>
      <div
        v-else-if="
          userData &&
          userData.liked_reviews &&
          userData.liked_reviews.length === 0
        "
      >
        <h3 class="text-center my-3">아직 추천한 리뷰가 없어요😯</h3>
      </div>
      <hr />
      <button
        class="click-btn"
        v-if="
          userData && userData.username && userData.username === store.userName
        "
        @click="leaveReelify(userData.username)"
      >
        Reelify 떠나기
      </button>
    </div>
  </div>
</template>

<script setup>
import { useAccountStore } from "@/stores/accounts";
import { useRoute, useRouter } from "vue-router";
import { onMounted, ref, watch } from "vue";
import axios from "axios";
import UserLikeMovie from "@/components/Community/UserLikeMovie.vue";
import UserReviewCard from "@/components/Community/UserReviewCard.vue";
import UserLikeReview from "@/components/Community/UserLikeReview.vue";
import UserRatingGraph from "@/components/Community/UserRatingGraph.vue";

import { RouterView } from "vue-router";

const store = useAccountStore();
const route = useRoute();
const router = useRouter();

// 새 데이터 정의
const userData = ref({});
const likeMovie = ref([]);
const likeMovieCnt = ref(0);
const likeReview = ref([]);
const likeReviewCnt = ref(0);
const writeReview = ref([]);
const writeReviewCnt = ref(0);
const profileImgUrl = ref("");
const userLikeGenre = ref([]);

const isFollow = ref(null);

const limitedReviews = ref([]);
const limitedLikeReviews = ref([]);

const loadUserData = (username) => {
  // 유저 정보 조회하는 요청
  axios({
    method: "get",
    url: `${store.API_URL}/api/v1/user-page/${route.params.username}/`,
  })
    .then((res) => {
      console.log(res.data);
      userData.value = res.data;
      likeMovie.value = res.data.liked_movies;
      likeMovieCnt.value = res.data.liked_movies.length;
      likeReview.value = res.data.liked_reviews;
      likeReviewCnt.value = res.data.liked_reviews.length;
      writeReview.value = res.data.written_reviews;
      writeReviewCnt.value = res.data.written_reviews.length;
      profileImgUrl.value = `${store.API_URL}${res.data.profile_img}`;

      // 리뷰가 3개 이상이면 3개만 가져오기
      limitedReviews.value =
        res.data.written_reviews.length > 3
          ? res.data.written_reviews.slice(0, 3)
          : res.data.written_reviews;

      limitedLikeReviews.value =
        res.data.liked_reviews.length > 3
          ? res.data.liked_reviews.slice(0, 3)
          : res.data.liked_reviews;

      console.log(limitedReviews.value);
    })
    .catch((err) => {
      console.log(err);
    });

  // 팔로잉 되어 있는지 아닌지 확인하는 요청
  axios({
    method: "get",
    url: `${store.API_URL}/api/v1/user/${route.params.username}/is_follow/`,
    headers: {
      Authorization: `Token ${store.token}`,
    },
  })
    .then((res) => {
      console.log(res.data);
      isFollow.value = res.data.is_following;
      console.log(isFollow.value);
    })
    .catch((err) => {
      console.log(err);
    });

  // 유저 선호 장르 데이터 요청
  axios({
    method: "get",
    url: `${store.API_URL}/api/v1/user-page/${route.params.username}/preferred_genres/`,
    headers: {
      Authorization: `Token ${store.token}`,
    },
  }).then((res) => {
    console.log(res.data);
    userLikeGenre.value = res.data;
  });
};

// 최초 로드
onMounted(() => {
  loadUserData(route.params.username);
});
// 라우트 변경 감지
watch(
  () => route.params.username,
  (newUsername) => {
    loadUserData(newUsername);
  }
);

// 파일 선택 트리거
const fileInput = ref(null);
const triggerFileInput = () => {
  fileInput.value.click();
};

// 이미지 업로드
const uploadProfileImage = async () => {
  const file = fileInput.value?.files[0];
  if (!file) {
    alert("파일을 선택해주세요.");
    return;
  }

  const formData = new FormData();
  formData.append("profile_img", file);

  try {
    const response = await axios({
      method: "patch",
      url: `${store.API_URL}/api/v1/profile-image/`,
      headers: {
        Authorization: `Token ${store.token}`,
        "Content-Type": "multipart/form-data",
      },
      data: formData,
    });

    profileImgUrl.value = `${response.data.profile_img}`;
    console.log(response.data.profile_img);
  } catch (error) {
    console.error("이미지 업로드 오류:", error);
    alert("이미지 업로드에 실패했습니다.");
  }
};

// 팔로우 구현
const followUser = function (username) {
  axios({
    method: "post",
    url: `${store.API_URL}/api/v1/user/${username}/follow/`,
    headers: {
      Authorization: `Token ${store.token}`,
    },
  })
    .then((res) => {
      isFollow.value = res.data.is_following;
      userData.value.followers_count += 1;
      console.log(isFollow.value);
    })
    .catch((err) => {
      console.log(err);
    });
};
// 팔로우 취소 구현
const unFollowUser = function (username) {
  axios({
    method: "post",
    url: `${store.API_URL}/api/v1/user/${username}/follow/`,
    headers: {
      Authorization: `Token ${store.token}`,
    },
  })
    .then((res) => {
      isFollow.value = res.data.is_following;
      userData.value.followers_count -= 1;
      console.log(isFollow.value);
    })
    .catch((err) => {
      console.log(err);
    });
};

// 유저 페이지 이동 함수
const goToUserPage = function (username) {
  router.push({ name: "UserPageView", params: { username: username } });
};

// 유저가 쓴 전체 리뷰 페이지 이동 함수
const goToUserReviewList = function (username) {
  router.push({ name: "UserReviewListView", params: { username: username } });
};

// 유저가 추천한 리뷰 리스트 페이지 이동 함수
const goToUserLikeReview = function (username) {
  router.push({ name: "UserLikeReviewView", params: { username: username } });
};

// 팔로워 목록 확인 페이지 이동 함수
const goToUserFollower = function (username) {
  router.push({ name: "UserFollowerView", params: { username: username } });
};
// 팔로잉 목록 확인 페이지 이동 함수
const goToUserFollowing = function (username) {
  router.push({ name: "UserFollowingView", params: { username: username } });
};

// 내 정보 수정 페이지 이동 함수
const goToUpdate = function (username) {
  router.push({ name: "UserUpdateView", params: { username: username } });
};

// 내 프로필 이미지 수정 페이지 이동 함수
const goToImageEdit = function (username) {
  console.log(userData.username);
  router.push({ name: "UserProfileEditView", params: { username: username } });
};

// 회원 탈퇴 기능
const leaveReelify = function (username) {
  const answer1 = window.confirm("정말 Reelify를 떠나실 건가요??😲");
  if (answer1 === true) {
    const answer2 = window.confirm("정말정말정말요??😱");
    if (answer2 === true) {
      const token = store.token;
      store.logOut();
      axios({
        method: "delete",
        url: `${store.API_URL}/api/v1/user/delete`,
        headers: {
          Authorization: `Token ${token}`,
        },
      })
        .then((res) => {
          console.log(res.data);
          window.confirm("그동안 Reelify를 사랑해주셔서 감사합니다.");
          router.push({ name: "HomeView" });
        })
        .catch((err) => {
          console.log(err.data);
          window.confirm("회원탈퇴 중 오류가 발생했습니다.");
        });
    }
  } else {
    return false;
  }
};
</script>

<style scoped>
.user-img {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  object-fit: cover;
  border: 5px solid #fba1b7;
  cursor: pointer;
}
.user-profile {
  margin-top: 70px;
  margin-bottom: 70px;
  margin-right: 150px;
  margin-left: 40px;
}
.profile-text {
  margin-right: 40px;
  margin-left: 40px;
}
.follow-text {
  font-size: 20px;
  cursor: pointer;
}
.username {
  font-weight: bold;
}
.update-btn {
  display: flex;
  color: white;
  background-color: #a1eebd;
  border-color: transparent;
  border-radius: 8px;
}
.click-btn {
  color: white;
  background-color: #a1eebd;
  border-color: transparent;
  border-radius: 8px;
}
.follow-btn {
  background-color: #f8f6e3;
  color: gray;
  border-color: transparent;
  border-radius: 8px;
  font-size: 20px;
}
.unFollow-btn {
  background-color: #a1eebd;
  color: white;
  border-color: transparent;
  border-radius: 8px;
  font-size: 20px;
}
.review-cnt {
  cursor: pointer;
  border-radius: 2px;
}
.review-cnt:hover {
  background-color: #a1eebd;
}
.likeReview-cnt {
  cursor: pointer;
  border-radius: 2px;
}
.likeReview-cnt:hover {
  background-color: #a1eebd;
}
.badge {
  background-color: aliceblue;
  color: black;
}
@media (max-width: 768px) {
  .profile-text {
    margin-left: 0;
    margin-right: 0;
  }

  .user-img {
    width: 100px;
    height: 100px; /* 화면이 작아지면 이미지 크기도 줄어듬 */
  }

  .follow-text {
    font-size: 1rem;
  }
}

@media (max-width: 576px) {
  .user-profile {
    flex-direction: column !important;
    text-align: center;
  }
  .follow-text {
    font-size: 0.9rem;
  }

  h3 {
    font-size: 1.2rem;
  }

  h1 {
    font-size: 1.5rem;
  }
}
</style>
