# Audio compatible con TV LG
[FFmpeg](https://ffmpeg.org/)

```sh
ffmpeg -i pelicula.mkv -c:v copy -c:a ac3 -b:a 640k pelicula_LG.mkv
```

