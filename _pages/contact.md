---
title: "Contact me!"
layout: single
excerpt: "Contact me"
sitemap: false
author_profile: true 
permalink: /contact/
---

<form action="https://formspree.io/blog@dair.io" method="POST" autocomplete="off">
    <fieldset>
        <input type="text" name="name" placeholder="name">
    </fieldset>
    <fieldset>
        <input type="email" name="_replyto" placeholder="email">
    </fieldset>
    <fieldset>
        <textarea type="text" name="text" placeholder="message"></textarea>
    </fieldset>
    <fieldset>
        <input type="submit" value="Send">
    </fieldset>
    <fieldset>
        <input type="hidden" name="_next" value="/thanks.html" />
        <input type="hidden" name="_subject" value="New submission to blog!" />
        <input type="text" name="_gotcha" style="display:none" />
    </fieldset>
</form>