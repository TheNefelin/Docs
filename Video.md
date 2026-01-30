## Fix audio for LG TV
[ffmpeg](https://ffmpeg.org/)

```sh
ffmpeg -i pelicula.mkv -c:v copy -c:a ac3 -b:a 640k pelicula_LG.mkv
```

---

