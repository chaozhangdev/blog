---
title: Uploading Videos & Generating Cover Image
date: 2022-03-05
description: Upload videos to the server with auto generating a cover image from the frontend.
---

![img](https://media.giphy.com/media/O1vudCHR4ZeTQtWr93/giphy.gif)

For the short video app, like TikTok, sometimes we need to upload our own videos. To improve the performance and user experiecne, we need a cover image(preload state). This requirement can be easily done in the native mobile app, but for the web app, I have a solution to realize it by just using javascript.

My solution is: get the video from the local disk => generate the temp url and play the video => resize the styles of this playing video by css and show it on the screen => get the screen shot of it by transfering the html to canvas => transfer canvas to form data and upload to the server

There are genral 4 steps including uploading the video to the server

### Step 0

Get the video from local disk, obtain the temp video url to play and transfer it to array buffer

```js
async function getBinaryDataFromFile(e) {
  let file = e.target.files[0]
  let videoURL = null
  let buffer = null
  if (file.type !== "video/mp4") {
    toast.error("Video must be .mp4 type.")
  } else {
    videoURL = URL.createObjectURL(file)
    buffer = await file.arrayBuffer()
    setTempVideoURL(videoURL)
    setBinaryFile(buffer)
  }
}
```

## Step 1

Get the screen shot of the playing video to generate the cover image which is the first frame of the video.

Here, I used a node module called [video-snapshot](https://www.npmjs.com/package/video-snapshot)

```js
async function getScreenShotFromVideo(e) {
  const file = e.target.files[0]
  const snapshoter = new VideoSnapshot(file)
  const previewSrc = await snapshoter.takeSnapshot()
  setVideoScreenShot(previewSrc)
}
```

## Step 2

Put the cover image into AWS S3 bucket first.

```js
const canvas = await html2canvas(exportRef.current)
const src = canvas.toDataURL("image/png", 1.0)
const fetchRes = await fetch(src)
const blob = await fetchRes.blob()
const time = new Date().getTime()
const file = new File([blob], `coverimage${time}.png`, blob)
const FormData = require("form-data")
const formData = new FormData()
formData.append("file", file)
const res = await request.post("video/uploadCover", formData)
const coverImgURL = res.data.code === 200 ? res.data.data.url : ""
```

## Step 3

Put the video into AWS S3 then, showing the real time percentage by the call back function in axios.

```ts
const res = await request.post("video/getUploadUrl")
const videoURL = res.data.data.url
let current = null
let total = null
let percent = null
await axios.put(videoURL, binaryFile, {
  headers: {
    "Content-Type": "video/mp4",
  },
  onUploadProgress: progressEvent => {
    current = progressEvent.loaded
    total = progressEvent.total
    percent = Math.floor((current / total) * 100)
    setLoadingPercentage(percent)
  },
})
```

"Content-Type": "video/mp4" is used to let the video in S3 playable instead of only downloadable

## Step 4

Commit uploading video with the cover image.

```js
const payload = {
  category_id: getCategoryIDbyName(),
  title: title,
  tags: tags === [] ? "" : tags.join(","),
  video_url: uploadURL,
  thumb: coverImgURL,
}

request
  .post("video/uploadCommit", payload)
  .then(res => {
    if (res.data.code === 200) {
      toast.success("success!")
    } else {
      console.log(res)
      toast.error("failure!")
    }
  })
  .catch(err => {
    console.log(err)
  })
```
