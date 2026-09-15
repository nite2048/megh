<script>
     const subject = new URL('./assets/subject.png', import.meta.url).href;
</script>


<main class="blur-panel vignette">


          <h2 class="header" style="color: wheat;">
               HAPPIEST BIRTHDAY 🍰🍰🍰🍰🍰
          </h2>

          <div class="hero retro-border" style="height: 70%; width: 80%;">
               <img src={subject} alt="duck">
          </div>

          <h2 class="subheader">
               keep slaying and uh doing lots of cool stuff, hope it doesnt come across as too creepy, enjoy your day and porasona korte thakis, youre totally awesome sauce 🦆🎃
          </h2>

</main>


<style>
     main {
          width: 100%;
          height: 100%;

          display: flex;
          justify-content: center;
          align-items: center;
          flex-direction: column;

          gap: 2rem;
     }

     img {
          display: block;
          width: 100%;
          height: 100%;
          flex-shrink: 0;
          object-fit: cover;
     }

     #top {
          display: flex;
          justify-content: center;
          align-items: center;
          gap: 1rem;
     }

     .retro-border {
          background: #c0c0c0;
          border: 0.5rem solid #000;
          box-shadow: 0.5rem 0.5rem 0 #000;
     }

     .pixel-bar {
          display: flex;
          align-items: center;
          padding: 0.5rem;
     }

     .pixel-input {
          flex: 1;
          min-width: 0;
          padding: 1rem;
          border: 0.25rem solid #000;
          background: #fff;
          outline: none;
     }

     .pixel-btn {
          background: #5555ff;
          color: #fff;
          border: 0.25rem solid #000;
          padding: 2rem 2rem;
          margin-top: 0;
          cursor: pointer;
          margin: 3rem;
     }

     .pixel-btn:active {
          background: #2222aa;
     }

     .sub {
          font-size: 1.2rem;
          font-weight: bold;
          color: #fff;
          text-align: center;
          text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.5);
     }

     .header {
          font-size: 2.4rem;
          font-weight: bold;
          margin-top: 2rem;

          color: greenyellow;
          text-align: center;
          text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.8);
     }

     .subheader {
          font-size: 1.2rem;
          font-weight: bold;
          color: #5555ff;
          text-align: center;
          text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.5);
     }

     :global(body) {
          height: 100vh;
          width: 100%;

          background: conic-gradient(
               violet 90deg,
               aqua 90deg 180deg,
               violet 180deg 270deg,
               aqua 270deg
          );

          background-repeat: repeat;
          background-size: 80px 80px;
          background-position: 0 0;

          animation-duration: 6s;
          animation-timing-function: linear;
          animation-iteration-count: infinite;
          animation-name: scroll-diag;
     }

     .vignette {
          --vignette-color: rgba(0, 0, 0, 0.3);
          --vignette-inner: rgba(0, 0, 0, 0.0);
          --border-radius: 12px;
          --padding: 1rem;

          position: relative;
          overflow: hidden;
          border-radius: var(--border-radius);
     }

     .blur-panel {
          --blur-size: 0.5px;

          background: rgba(255, 255, 255, 0.12);

          backdrop-filter: blur(var(--blur-size));
          -webkit-backdrop-filter: blur(var(--blur-size));

          z-index: 2;
     }

     .vignette::before {
          content: "";
          position: absolute;
          inset: 0;
          pointer-events: none;

          background-image: radial-gradient(
               circle at center,
               var(--vignette-inner) 50%,
               var(--vignette-color) 100%
          );

          mix-blend-mode: multiply;
          z-index: 5;
     }

     .vignette > .content {
          position: relative;
          z-index: 10;
          color: #fff;
     }
</style>