<div class="skeleton">
  <div class="children">
  </div>
</div>

<style>
  .skeleton {
    height: 40%;
    width: 80%;
    margin-left: 10%;
    margin-right: 10%;
    align-self: center;
    opacity: 0;
    display: flex;
    align-items: stretch;
    justify-content: stretch;
    background-color: var(--spectrum-global-color-gray-200) !important;
    border-radius: 7px;
    overflow: hidden;
    position: relative;
    animation: fadeIn 130ms ease 0s 1 normal forwards;
  }
  .children {
    flex: auto;
    pointer-events: none;
    opacity: 0.85;
    background-image: linear-gradient(
      90deg,
      rgba(255, 255, 255, 0) 0,
      rgba(255, 255, 255, 0.15) 20%,
      rgba(255, 255, 255, 0.3) 60%,
      rgba(255, 255, 255, 0)
    );
    animation: shimmer 2s infinite;
  }

  @keyframes fadeIn {
    0% {
      opacity: 0;
    }
    100% {
      opacity: 0.75;
    }
  }
  @keyframes shimmer {
    100% {
      transform: translateX(100%);
    }
  }
</style>