+++
title = "MYOX: Youtube downloader"
description = "Lets implement Youtube downloader"
+++

## Preface

While walking, training, going to work I like to listen to various talks, podcasts, etc. Recently I decided to save a couple of them on my phone. I found some web apps which allowed me to do that. But there were some many ads, it was so unpleasant to deal with them. This is why started to looking for a way how I can do that on my own, without intermediate services. And as you will see in a few minutes it's easy to do.  

I will show how to implement a trivial downloader(a possibility to dynamically select a resolution, download only audio tracks - their implementation is trivial and you will be able to implement it on your own, as an excercise if you will).

## Implementation

It will be a binary create since we are going to use it as a usual application.
My intention is to run it as:
```bash
./youtube-downloader https://www.youtube.com/watch?v=Bn40gUUd5m0
```

Lets create a project(in order to add packages I'll use [cargo add](https://github.com/killercup/cargo-edit)):

```bash
cargo new youtube-downloader
cd youtube-downloader

# flexible Results/Errors
cargo add anyhow 
# parse id
cargo add regex 
# network requests
cargo add reqwest --features blocking
# traverse querystring
cargo add qstring 
# payload deserialization
cargo add serde_json 
```

All stuff will be placed in one ~main.rs~ file.  

```rust
use anyhow::Result;
use qstring::QString;
use reqwest::{blocking::Client, Url};
use serde_json::Value;

// read the passed by a user link
use std::env;
```

Firstly we need to fetch an id from link which a user passed. Lets begin from a test:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_id_from_url_extraction_short() {
        let url = "https://www.youtube.com/watch?v=Bn40gUUd5m0";
        let id = get_video_id(url);

        assert!(id.is_some());
        assert_eq!(id.unwrap(), "Bn40gUUd5m0");
    }
}
```

As you can see the video id is placed behind ~?v=~.
We won't be very attentive, such as handling extended with params, but valid links.

Implementation:

```rust
fn get_video_id(url: &str) -> Option<&str> {
    if let Some(id) = regex::Regex::new(r"https://www\.youtube\.com/watch\?v=(.*)")
        .expect("correct regex")
        .captures(url)
        .unwrap()
        .get(1)
    {
        Some(id.as_str())
    } else {
        None
    }
}
```

Now when we have the video id we can ask youtube to give us an info about that video file. We can do that via the special api endpoint which youtube provides.
A simple test just to be sure that something was fetched:

```rust
   #[test]
   fn test_get_video_info() {
       let url = "https://www.youtube.com/watch?v=zCLOJ9j1k2Y";

       let info = get_video_info(url);

       assert!(info.is_ok());
   }
```

```rust
fn get_video_info(url: &str) -> Result<Value> {
    let json = if let Some(id) = get_video_id(url) {
        let video_url = format!(
            "https://www.youtube.com/get_video_info?video_id={}&el=embedded&ps=default",
            id
        );
        let res_body = reqwest::blocking::get(video_url.as_str())?.text()?;

        QString::from(res_body.as_str())
            .get("player_response")
            .unwrap_or("")
            .to_owned()
    } else {
        return Err(anyhow::Error::msg("couldn't get video id".to_string()));
    };

    
    Ok(serde_json::from_str(&json)?)
}
```

Explanation: we fetch the video info via the special endpoint. After that we are trying to find **player_response** part from query string. The placed there value is json, so if it's present we try to parse it. I encourage you to explore that parsed object.  
In the parsed object there is a lot of information. We are interesting in **video_info["streamingData"]["formats"]**. There we can find various video formats with their metainfo. Also there is **video_info["streamingData"]["adaptiveFormats"]** part where you can find even more available to download formats. All those objects have a field **url** which describes the actual place from which we can extract the video. We will focus on **video_info["streamingData"]["formats"]** since it's more simple and it's ok for this tutorial.  
So lets move on implement a function which should give us an actual download link. As I said earlier we won't dig into dynamic video quality configuration, just download the most simple one.


```rust
    #[test]
    fn test_get_video_download_url() {
        let url = "https://www.youtube.com/watch?v=zCLOJ9j1k2Y";
        let video_info = get_video_info(url).unwrap();
        let url = get_video_download_url(&video_info);

        assert!(url.is_some());
    }
```

```rust
fn get_video_download_url(video_info: &serde_json::Value) -> Option<&str> {
    // since there are many formats and their codec we are going to download the post popular
    let mp4_codec_regex = regex::Regex::new(r"codecs=(.*mp4.*)").expect("correct codecs regexp");
    for t in video_info["streamingData"]["formats"].as_array().unwrap() {
	    // so that we can check the appropriate needed video quality
        if let Some("360p") = t["qualityLabel"].as_str() {
            if mp4_codec_regex
                .find(t["mimeType"].as_str().unwrap())
                .is_some()
            {
                return Some(t["url"].as_str().unwrap());
            }
        }
    }
    None
}
```

Now, when we can fetch the actual link to the file we can try to download it. I will write a simple test just to not run an app manually if something was wrong.

```rust
    #[test]
    fn test_file_download() {
        let url = "https://www.youtube.com/watch?v=Bn40gUUd5m0";
        let video_info = get_video_info(url).unwrap();
        let url = get_video_download_url(&video_info);
        let file_name =
            get_video_file_name(&video_info).expect("filename in video_info is present");

        download_file(url.unwrap(), &file_name);

        assert!(Path::new(&file_name).exists());
    }
```

In order to fulfill that final test we need to somehow create filenames. I suggest to just pick it up from the video info.

```rust
fn get_video_file_name(video_info: &Value) -> Option<String> {
    if let Some(name) = video_info["videoDetails"]["title"].as_str() {
        Some(format!("{}.mp4", name))
    } else {
        None
    }
}
```

Finally the actual download function implementation: 

```rust
fn download_file(url: &str, file_name: &str) -> Result<()> {
    let url = Url::parse(url)?;
    let mut resp = Client::new().get(url.as_str()).send()?;
    let mut out = std::fs::File::create(file_name)?;
    std::io::copy(&mut resp, &mut out)?;

    Ok(())
}
```

Here we make a simple get request by the found download link and after that save a contents to a file.
Seems works. Great!  

Finally lets finish our main function:

```rust
fn main() -> Result<()> {
    let args: Vec<String> = env::args().collect();
    if let Some(link) = args.get(1) {
        let video_info = get_video_info(link)?;
        let url = get_video_download_url(&video_info).expect("video download url found");
        let file_name =
            get_video_file_name(&video_info).expect("filename in video_info is present");
        download_file(url, &file_name[..])?;

        Ok(())
    } else {
        Err(anyhow::Error::msg(
            "video link must be provided".to_string(),
        ))
    }
}
```

And now lets test it how we can use an app from the cli:

```bash
cargo run https://www.youtube.com/watch?v=Bn40gUUd5m0
```

After that you can see the downloaded file in the project root folder.

Also we can do:

```bash
cd target/debug
./youtube-downloader https://www.youtube.com/watch?v=Bn40gUUd5m0
```
So if you want to be able to use it from any location you need to do:

```bash
ln -s <path-to--your-executable> ~/bin
# after that 
youtube-downloader https://www.youtube.com/watch?v=Bn40gUUd5m0
# but be careful since now it downloads the file relative to the location
# so probably if you want use it in such way you need to implement an additional arg
# which specifies a path where the file should be stored
```

## Conclusion

As you see the implementation is trivial and could be done in an hour including googling.

You can add more features such as:
 - select the quality
 - select video or only audio
 - show a progress bar while downloading
 - batch save when needed to save many videos
