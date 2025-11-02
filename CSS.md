# CSS

```css
html {
  scroll-behavior: smooth;
}
```

```css
/* Chrome, Safari */
body::-webkit-scrollbar{
  display: none !important;
}

html::-webkit-scrollbar{
  display: none !important;
}
```

```css
/* Para Firefox */
body {
  scrollbar-width: none !important; 
}

html {
  scrollbar-width: none !important; 
}
```

---

```html
<section>
  <header> Header </header>
  <main> Main </main>
  <footer> Footer </footer>
</section>
```
```css
section {
  display: grid;
  min-height: 100dvh;
  grid-template-rows: auto 1fr auto;
}
```