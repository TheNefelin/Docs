# Framework NextJS

* [NextJS](https://nextjs.org/)
* [Tailwind css](https://tailwindcss.com/)
## 
* Virtual DOM
* Browser DOM: arbol de navegacion de una web
<html> -> <head> <body>

```mermaid
graph TD;
  App
  <html> --> <head>
  <html> --> <body>     
  <head> --> <link>
  <head> --> <meta>    
  <body> --> <nav>
  <body> --> <main>
  <body> --> <footer>        
  ;
```
### Instalacion
* TypeScript
* ESLint
* Taildwind
* src
* App Router
* Turbopack
* import alias

>[alert!] alternativa a npm es yarn

### Base
* react
* react-dom
* next

###
* Rub
* Build
```
npm run dev
npm run build
```

# Condicionales
```
export default async function NommbreFuncion() {
  const fetching = new Fetching()
  const apiResult: ApiResult = await fetching.get_all_games_async()

  return(
    {apiResult.isSucces? (
    <>
      <Componente_1/>
      {children}               
    </>
    ) : (
    <>
      <Componente_2>
      <Componente_3>
      <Componente_4>
    </>
    )}
  )  
}
```
```
export default async function NommbreFuncion() {
  const fetching = new Fetching()
  const apiResult: ApiResult = await fetching.get_all_games_async()

  return(
    {data.map((game:SideBarData) => (
      game.isActive? (
        <Componente_1>
      ) : (
        <>
          <Componente_2>
          <Componente_3>
          <Componente_4>
        </>
      )  
    ))}
  )  
}
```

# Pasar Datos a hijos
```
interface SideBarProps {
  data: MyType[]
}

export default function NommbreFuncion({data}: SideBarProps) {

  return (
    {data.map((game:MyType) => (
      game.isActive? (
        <li key={game.id} className='mb-2'>
          <p>{game.name}</p>
        </li>
      ) : (
        <></>
      )  
    ))}
  )
}
```
```
import React, { FC } from 'react';

const NommbreFuncion: FC<{data: MyType[]}> = ({data}) => {
  return (
    <ul>
      {data.map((game: MyType) => {
        if (!game.isActive) return null;  // No renderizar nada si no está activo
        return (
          <li key={game.id} className="mb-2">
            <p>{game.name}</p>
          </li>
        );
      })}
    </ul>
  );
}

export default NommbreFuncion;
```