---
title: Develop a Tiktok like Web App
date: 2022-03-07
description: Develop a short videos web application from scratch by react.
---

![img](https://media.giphy.com/media/JfqBvJgsbwrchWYR5w/giphy.gif)

![img](https://media.giphy.com/media/kiOcCyjSvgJ0qd82OK/giphy.gif)

Node modules:

[react-infinite-scroll-component](https://www.npmjs.com/package/react-infinite-scroll-component): view the videos infinitely

[react-visibility-sensor](https://www.npmjs.com/package/react-visibility-sensor): auto play or resume videos during scrolling

[react-player](https://www.npmjs.com/package/react-player): react video player

```jsx
<InfiniteScroll
  dataLength={data.length}
  next={fetchMoreData}
  hasMore={true}
  loader={
    <div className="text-center d-flex flex-row justify-content-center">
      <ReactLoading
        type="bubbles"
        color="#e6005a"
        height={"100%"}
        width={160}
      />
    </div>
  }
>
  {data !== [] &&
    data.map((el: any, index: number) => (
      <VisibilitySensor
        key={index}
        onChange={(status: boolean) => {
          status && setCurrentPlayingVideoIndex(index)
        }}
      >
        <div>
          <AllWrapper>
            <LeftWrapper>
              <ReactPlayer
                loop={true}
                controls={true}
                playing={index === currentPlayingVideoIndex}
                url={el.video_url}
                width="100%"
                height="500px"
                onPlay={() => {
                  handleView(index)
                }}
              />
            </LeftWrapper>
            <RightWrapper>
              <RightTopBar>
                <Link to={`/uploader/${el.user_id}`}>
                  <UserWrapper>
                    <UserAvatar src={el.avatar} alt="avatar" />
                    <Text className="mx-4">{el.nickname}</Text>
                  </UserWrapper>
                </Link>
                {el.is_follow === 0 ? (
                  <button
                    className="mybtn"
                    onClick={() => checkIsLogin() && handleFollow(el.user_id)}
                  >
                    Follow
                  </button>
                ) : (
                  <button
                    className="mybtn-grey"
                    onClick={() => checkIsLogin() && handleUnFollow(el.user_id)}
                  >
                    Unfollow
                  </button>
                )}
              </RightTopBar>
              <Text>
                <Icon>💻</Icon> Title: {el.title}
              </Text>
              <Link to={`/category/${el.category_id}@${el.category_name}`}>
                <Text>
                  <Icon>📦</Icon> Category: {el.category_name}
                </Text>
              </Link>
              <Text>
                <Icon>⏰</Icon> Time: {el.created_at.split(" ")[0]}
              </Text>
              <div className="d-flex flex-sm-column flex-row flex-wrap">
                <Function>
                  <p>View</p>
                  <img src={`./images/video-functions/view.png`} alt="png" />
                  <p>{el.view_num}</p>
                </Function>
                <Function>
                  <p>Like</p>
                  <img
                    className={
                      el.is_like === 1
                        ? "animate__animated animate__bounceIn"
                        : ""
                    }
                    src={
                      el.is_like === 1
                        ? "./images/video-functions/like1.png"
                        : "./images/video-functions/like.png"
                    }
                    alt="png"
                    onClick={() => checkIsLogin() && handleLike(index)}
                  />
                  <p>{el.like_num}</p>
                </Function>
                <Function>
                  <p>Comment</p>
                  <img
                    onClick={() => {
                      checkIsLogin() && navigate(`videoplayer/${el.id}`)
                    }}
                    src="./images/video-functions/comment.png"
                    alt="png"
                  />
                  <p>{el.comment_num}</p>
                </Function>
                <Function>
                  <p>Share</p>
                  <img
                    onClick={() => checkIsLogin() && handleShare()}
                    src="./images/video-functions/sharing.png"
                    alt="png"
                  />
                  <p>{el.share_num}</p>
                </Function>
              </div>
              <TagsWrapper>
                {el.tags !== "" &&
                  el.tags.split(",").map((str: string) => (
                    <Link to={`/tags/${str}`} key={str}>
                      <p className="cursor-pointer">{"# " + str}</p>
                    </Link>
                  ))}
              </TagsWrapper>
            </RightWrapper>
          </AllWrapper>
          <VideoDivider></VideoDivider>
        </div>
      </VisibilitySensor>
    ))}
</InfiniteScroll>
```
