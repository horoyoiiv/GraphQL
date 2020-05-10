# GraphQL
grapQL

### [1. Kakao GraphQL](https://tech.kakao.com/2019/08/01/graphql-basic/)  
### [2. GraphQL tutorial (very good)](https://www.youtube.com/watch?v=Y0lDGjwRYKw&list=PL4cUxeGkcC9iK6Qhn-QLcXCXPQUov1U7f)  
### [3. 심플하게 설명](https://medium.com/@yeon22/graphql-graphql%EC%9D%B4%EB%9E%80-8468571ea96a)  
### [4. REST vs GraphQL](https://www.holaxprogramming.com/2018/01/20/graphql-vs-restful-api/)  


## 1. GraphQL이란?  
 * facebook에서 개발한 SQL과 같은 데이터에 대한 **쿼리 언어**이다.   
 * 하지만 SQL은 **어플리케이션**에서 **DB**로 질의하는 것이라면  
 GQL은 **웹 클라이언트**가 **서버**로 질의할 때 사용하는 쿼리 언어.  
 
## 2. "웹 클라이언트 -> 서버"  
* 그렇다면 웹 클라이언트에서 서버로 질의하는 또 다른 방식인 **REST API**와는 어떤 차이가 있는지.  

## 3. 단점...  
* 주고 받는 데이터 형식이 JSON이라면 **이진 파일**을 어떻게 주고 받을 수 있을까?   


#### 2.1. single endpoint  
* REST API는 **URL**을 통해 자원을 식별한다. 그렇기에 **여러 엔드포인트**가 서버에 생긴다.  
* GraphQL은 "/graphql"이라는 싱글 포인트를 사용한다.  
이는 프론트 개발자가 추가적인 데이터가 필요할 경우, 백엔드 개발자에게 또 다른 api를 파달라고 요청하지 않아도 된다.  

#### 2.2. over 혹은 under fetching이 없다.  
* 클라이언트에서 필요한 데이터만큼을 정확히 가져올 수 있다.  


## 2. Why Graph인가?  
* GraphQL에서는 **모델 간의 관계**를 **그래프**로 설계하기 때문이다.    
![image](https://user-images.githubusercontent.com/62331555/81508333-60d44180-933e-11ea-9050-543144395037.png)  

* GraphQL에서 오브젝트 타입의 필드가 **스칼라**가 아닌, 사용자가 정의한 또 다른 오브젝트 타입인 경우 또 **리졸버**를 호출한다.  
* 이처럼 **루트 쿼리**로부터 쿼리가 리졸버에 의해 연쇄적으로 발생하기에, 이는 마치 Graph에서의 DFS와 유사해서...  



## 3. 용어  

```gql
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```
* Character : **오브젝트 "타입"**  
* name, appearsIn : **필드**  
* 스칼라 타입 : String, ID, Int, Boolean  
* !(느낌표) : 필수  
* [] : 배열  



## 4. 리졸버  
#### "리졸버"는 실제로 데이터 소스와 Interaction하는 곳.  
#### resolve 함수 안에서 데이터를 실제 쿼리하여, 반환한다.  

* **gql**에 대한 쿼리문 파싱은 대게 gql 라이브러리가 해준다.  
* 하지만 실제 해당 요청에 대한 데이터 조회는 개발자가 **리졸버 내에서** 직접 구현해야 한다.  

* 이러한 리졸버는 resolve 함수 안에서 **개발자가 직접 쿼리문을 작성해야 한다는 부담**이 있다.  
### 킹치만 이는 데이터 소스의 자유도가 높아진다. RDBMS, NoSQL 심지어 다른 REST API로 요청이 가능하다...!  

## 5. 리졸버 사용  

1. gql 쿼리에서는 **각각의 필드마다 함수가 하나씩 존재 한다**고 생각하면 됩니다.  
2. 이 함수는 다음 타입을 반환합니다. 이러한 각각의 함수를 **리졸버(resolver)**라고 합니다.  
3. 만약 필드가 스칼라 값(문자열이나 숫자와 같은 primitive 타입)인 경우에는 실행이 종료됩니다. 
4. 즉 더 이상의 연쇄적인 리졸버 호출이 일어나지 않습니다.  
5. **하지만 필드의 타입이 스칼라 타입이 아닌 우리가 정의한 타입이라면 해당 타입의 리졸버를 호출되게 됩니다.**  


```js
const BookType = new GraphQLObjectType({        // book type을 정의한다.
  name : 'Book',
  fields: () => ({
    id : {type : GraphQLID},
    name : {type : GraphQLString},
    genre : {type : GraphQLString},
    author : {                                  // 타입의 필드가 스칼라 타입이 아닌, 우리가 정의한 타입이면 연쇄적으로 리졸버를 호출함...!
      type : AuthorType,                        // 이 필드의 타입을 알려주고
      resolve(parent, args){                    // 리졸버를 정의해준다.
        
        // 1) mysql에서 쿼리해온다.
        // 2) mongoDB에서 쿼리해온다.
        // 3) rest api로 쿼리해온다.
        dataset = parent.authorid           // 관계에 대한 criteria 적용 가능.
        return 그 결과 반환
      }
    }
  })
});
```


```gql
.../graphql
// query 
{
  book(id : 2){     // bookType의 객체 반환 - id = 2인것     
    name            // 그중 필드가 name, genre를 반환
    genre
    author{         // 또한 book과 관계를 맺고 있는 authorType을 반환 
      name          // 그 중 필드 name 만 반환
    }
  }
}
```

```gql
반환값.
{
  "data" : {
    "book" : {
      "name" : "Hello world",
      "genre" : "Fantasy",
      "author" : {
        "name" : "hugo"
      }
    }
  }
}
```

## 6. Query 와 Mutation  

* 이는 클라이언트의 **Query**문을 처리하기 위함.  
```js
const RootQuery = new GraphQLObjectType({
  name : "RootQueryType",
  fields : {}
});
```

* 이는 클라이언트의 **mutation**을 처리하기 위함.  

```js
const Mutation = new GraphQLObjectType({
  name : "Mutation",
  fields : {
    
    addAuthor : {                             // Mutation 요청 시 사용될 이름
      type : AuthorType,
      
      args:{
        name : {type : GraphQLString},
        age : {type : GraphQLInt}
      },
      
      resolve(parent, args){                // addAuthor 라는 mutation 요청 시 실행할 resolver 함수 등록
        let author = new Author({             // author 객체 생성 후
          name : args.name,
          age : args.age
        });                           
        return author.save();                 // db에 save()
      }                                         // resolve 함수 끗
    }
  }
})
```

```gql
mutation{
  addAuthor(name : "steven", age : 38){     // 이러한 인자를 가지는 author를 생성하세요
     name                               // 글고 생성 완료 시 name, age를 다시 조회해주세요.
     age
  }
}


반환값.
{
  "data" : {
    "addAuthor" : {
      "name" : "steven",
      "age" : 38
    }
  }
}
```

## 7. Query Variables  

* gql 외부에서 동적으로 데이터를 추가할 때...  
```gql
mutation{
  addBook(name : "hello", genre : "sifi", authorId : 1){     
     name                              
     genre
  }
}
```
* 동적으로 추가하기 위하여...  

```gql
mutation ($name : String!, $genre: String!, $authorId: ID!) {   // $ 달러사인은 해당 이름으로 변수가 들어올 것을 의미 : 타입
  addBook(name : $name, genre : $genre, authorId : $authorId){   // addBook의 필드인 name에는 이 뮤테이션 안에서 사용되는 쿼리 변수인 $name을 대입.
     name   // 완료 시 해당 값을 반환                              
     genre
  }
}
```

  












