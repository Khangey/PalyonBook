<template>
  <div class="opds-page">
    <h1>OPDS ངོ་སྤྲོད།</h1>
    <section>
      <h2>OPDS དྲ་ཐལ</h2>
      <p>ཁྱོད་ཀྱི OPDS དྲ་ཐག་ནི་<code>{{ opdsUrl }}</code></p>
    </section>
    <section>
      <h2>རྒྱུན་མཐོང་གི་OPDSལྟ་ཀློག་མཉེན་ཆས།</h2>
      <ul>
        <li><strong>KyBook</strong>ནི་iOS དང macOS ལ་འཚམ་པའི་ལྟ་ཀློག་ཆས། <a href="https://apps.apple.com/app/kybook/id1049898139" target="_blank" rel="noopener noreferrer">[ཕབ་ལེན།]</a></li>
        <li><strong>静读天下</strong>ནི་Android ལ་འཚམ་པའི་ལྟ་ཀློག་ཆས། <a href="https://play.google.com/store/apps/details?id=com.flyersoft.moonreader" target="_blank" rel="noopener noreferrer">[ཕབ་ལེན།]</a></li>
      </ul>
    </section>
    <section>
      <h2>སྒྲིག་འགོད་ལས་རིམ།</h2>
      <ol>
        <li>ལྟ་ཀློག་ཆས་ཀྱི་ཁྲོད་དུ་ OPDS དཔེ་མཛོད་བསྣན་རོགས།/li>
        <li>ཁྱོད་ཀྱི་ OPDS དྲ་ཐག་བསྐོང་དགོས།</li>
        <li>དྲན་སྐུལ་ལྟར་དཔང་འཛིན་གནང་རོགས།</li>
        <li>ཁྱེད་ཀྱི་དཔེ་ཆ་ལ་ལྟ་ཀློག་བགྱིས་དང་།</li>
      </ol>

      <h2>དྲན་སྐུལ།</h2>
      <p>静读天下APPནསcookieཐོ་འགོད་ལ་རྒྱབ་སྐྱོར་མི་ཐོབ་པས། གཤམ་ལྟར་སྒྲིག་འགོད་གཏོང་དགོས།</p>
      <ol>
          <li>སྒེར་གྱི་རྣམ་པ་ཁ་རྒྱག་དགོས་ལ།</li>
          <li>ཐོ་འགོད་མ་བྱེད་པར་ཕབ་ལེན་གྱི་ཆོག་མཆན་ཐོབ་པའི་བྱེད་ནུས་ཁ་ཕྱེས་དགོས།</li>
      </ol>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      opdsUrl: window.location.origin + '/opds/'
    }
  }
}
</script>

<style scoped>
.opds-page {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

section {
  margin-bottom: 30px;
}

h2 {
  margin-bottom: 15px;
}

ul, ol {
  padding-left: 20px;
}

code {
  background: #f5f5f5;
  padding: 2px 5px;
  border-radius: 3px;
}
</style>
