---
layout: post
title: Remove audio from video files
comments: true
---

Simple one liner to remove audio from video (useful for private videos which have crappy audio):

```
ffmpeg -i ${INPUT} -vcodec copy -an ${OUTPUT}
```

The magic is triggered by the `-an` option.
