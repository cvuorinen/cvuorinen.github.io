---
layout: page
title: Contact
permalink: /contact/
---

<form action="https://formspree.io/carl.vuorinen@gmail.com"
      method="POST">
    <div>
        <label>Name *</label>
        <input type="text" name="name" required>
    </div>
    <div>
        <label>Email *</label>
        <input type="email" name="_replyto" required>
    </div>
    <div>
        <label>Comment *</label>
        <textarea name="comment" rows="10" required></textarea>
    </div>
    
    <input type="submit" value="Send">
    
    <input type="hidden" name="_subject" value="Contact form submission from cvuorinen.net" />
    <input type="text" name="_gotcha" style="display:none" />
</form>
