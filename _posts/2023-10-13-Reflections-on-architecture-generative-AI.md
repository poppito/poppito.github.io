---
slug: '2023-10-13-Reflections-on-architecture-generative-AI'
title: 'Reflections on the architectural approach for my first generative AI app.'
date: 2023-10-13 10:30 +1100
categories: 'blog'
---

# User safety above all else.

Some reflections on the architecture of shipping my first ever generative API + writing a backend for it:

1) User safety above all else. Generative AI is awful as far as bias is concerned. The API I chose allows tweaking the safety settings for every endpoint ☢️

More [info](https://developers.generativeai.google/guide/safety_setting#safety_settings_2)

2) Prompt engineering is real! cURL to the rescue. But wrapping user interaction calls in my own endpoints also gives me full control over tweaking prompts 🛟

3) A/B testing is an added advantage of 2). No user identifiable data is ever collected to create A/B testing cohorts, just UUIDs for each session 🕵️‍♀️

4) Adding an extra layer of security by adding a force-upgrade path for each version of the app. ⬆️

5) Did you notice that OpenAI killswitch meme? Yeah, I built it in 4) because user safety above all else and especially in case of 1) going awry 🎛️

6) Using SwiftUI was seamless to build for universal apps (iPad, iPhone, MacOS…and visionPro one day) 🌍

7) Swift Charts in SwiftUI because 6) 📈 🤩

8) PalmAPI isn’t available everywhere in the world, and most notably in a lot of European countries, and neither is the app. Highlighting this to the app store review peeps eventually got it approved (after about 4 rejections, 1 expedited review and me personally paying for a proxy to include in my app store review) 📝

thanks for reading 🙏🏾