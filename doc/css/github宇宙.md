```vue
<template>
  <div class="bg-black h-full">
    <div class="signup-space">
      <div class="signup-stars"></div>
      <div class="signup-stars"></div>
      <div class="signup-stars"></div>
      <div class="signup-stars"></div>
      <div class="signup-stars"></div>
      <div class="signup-stars"></div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.signup-space,
.signup-stars {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  overflow: hidden;
}

.signup-stars {
  background-image: radial-gradient(
      2px 2px at 50px 200px,
      #eee,
      rgba(0, 0, 0, 0)
    ), radial-gradient(2px 2px at 40px 70px, #fff, rgba(0, 0, 0, 0)),
    radial-gradient(3px 4px at 120px 40px, #ddd, rgba(0, 0, 0, 0));
  background-repeat: repeat;
  background-size: 250px 250px;
  opacity: 0;
  animation: zoom 10s infinite;
}

.signup-stars:nth-child(1) {
  background-position: 10% 90%;
  animation-delay: 0s;
}

.signup-stars:nth-child(2) {
  background-position: 20% 50%;
  background-size: 270px 500px;
  animation-delay: 0.3s;
}

.signup-stars:nth-child(3) {
  background-position: 40% -80%;
  animation-delay: 1.2s;
}

.signup-stars:nth-child(4) {
  background-position: -20% -30%;
  transform: rotate(60deg);
  animation-delay: 2.5s;
}

.signup-stars:nth-child(5) {
  background-image: radial-gradient(
      2px 2px at 10px 100px,
      #eee,
      rgba(0, 0, 0, 0)
    ), radial-gradient(2px 2px at 20px 10px, #fff, rgba(0, 0, 0, 0)),
    radial-gradient(3px 4px at 150px 40px, #ddd, rgba(0, 0, 0, 0));
  background-position: 80% 30%;
  animation-delay: 4s;
}

.signup-stars:nth-child(6) {
  background-position: 50% 20%;
  animation-delay: 6s;
}

@keyframes zoom {
  0% {
    opacity: 0;
    transform: scale(0.5);
    transform: rotate(5deg);
    animation-timing-function: ease-in;
  }

  85% {
    opacity: 1;
  }

  100% {
    opacity: 0.2;
    transform: scale(2.2);
  }
}

@media (prefers-reduced-motion) {
  .signup-stars {
    animation: none;
  }
}

.signup-warp-bg {
  width: 100vw;
  height: 100vh;
  margin: 0;
  visibility: hidden;
  background: transparent url("/images/modules/signup/launch_codes/bg.jpg") 50%
    50% no-repeat;
  background-size: cover;
  opacity: 0;
}

.signup-warp-bg.postwarp {
  visibility: visible;
  opacity: 1;
  transition: 0.5s opacity ease-in, 0.5s visibility ease-in;
}

.signup-warp-bg.skipwarp {
  visibility: visible;
  background: #fff;
  opacity: 1;
  transition: 1.5s background ease-in, 1.3s opacity ease-in,
    1.3s visibility ease-in;
}

.signup-warp-vid {
  position: absolute;
  bottom: 25vh;
  left: 25vw;
  display: flex;
  flex-flow: row wrap;
  width: 50vw;
  height: 50vh;
  visibility: hidden;
  opacity: 0;
  align-content: center;
  justify-content: center;
}

.signup-warp-vid .signup-warp-video {
  width: 100%;
  height: 62%;
  margin: auto;
}

.signup-warp-vid.postwarp {
  visibility: visible;
  opacity: 1;
  transition: 1.5s opacity ease-in 3s, 1.5s visibility ease-in 3s;
}

.signup-warp-vid.skipwarp {
  visibility: visible;
  opacity: 1;
  transition: 2.2s opacity ease-in 1.2s, 2.2s visibility ease-in 1.2s;
}

.warp-hide {
  visibility: hidden;
  opacity: 0;
  transition: 0.5s opacity ease-in, 0.5s visibility ease-in;
}

.mona-fallback {
  width: 50vw;
}

/*# sourceMappingURL=signup-96cd9524fe80.css.map*/
</style>
```
