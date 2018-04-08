---
layout: page
title: Contact
permalink: /contact/
---

<form name="contact" method="POST" netlify>
    <div>
        <label>Name *</label>
        <input type="text" name="name" autocomplete="name" required />
    </div>
    <div>
        <label>Email *</label>
        <input type="email" name="email" autocomplete="email" required />
    </div>
    <div>
        <label>Comment *</label>
        <textarea name="comment" rows="10" required></textarea>
    </div>

    <input type="submit" value="Send" />
</form>
