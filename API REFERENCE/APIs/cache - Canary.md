<aside>
💡  <code>cache</code> 는 <strong>Server Components</strong> 에서만 사용 가능하다. - framework 를 사용해야한다.

</aside>

### Reference

- `cache` 는 데이터 페치나 계산의 결과를 캐시해준다.
    
    ```jsx
    import {cache} from 'react';
    import calculateMetrics from 'lib/metrics';
    
    const getMetrics = cache(calculateMetrics);
    
    function Chart({data}) {
      const report = getMetrics(data);
      // ...
    }
    ```
    
    - 컴포넌트 밖에서 캐시된 함수 버전을 생성해낸다.

### Parameters

- <code>**cache(fn)**</code>
    - `fn` : 결과를 캐시하려는 함수. 모든 인수를 받고 모든 값을 반환할 수 있다.

### Returns

- `cache` 는 `fn` 의 캐시된 같은 타입을 반환한다.
- `fn` 를 사용해 cachedFn 을 호출하면 캐시된 결과가 있는지 확인한다. 없으면 `fn` 을 호출하고 결과를 캐시에 저장한 후 결과를 반환한다.
( <code>**fn**</code> 이 호출되는 유일한 경우는 **cache miss** 일 때 뿐이다. )

### Caveats

- React 는 각 서버 요청에 대해 메모된 모든 함수에 대한 캐시를 무효화한다.
- 각 `cache` 함수는 새로운 함수를 생성시킨다. 즉, 같은 함수로 캐시를 여러 번 호출하면 매 번 새로운 메모화된 함수가 반환된다.
- cachedFn 는 오류도 캐시한다. `fn` 이 특정 인수에 대해 에러를 발생시키면, 이를 캐시하고, 동일한 인수로 cachedFn 을 호출할 때 동일한 에러가 다시 발생한다.
- `cache`는 서버 컴포넌트만 사용할 수 있다.

### Usage

- 비싼 계산 캐시하기
    
    ```jsx
    import {cache} from 'react';
    import calculateUserMetrics from 'lib/user';
    
    const getUserMetrics = cache(calculateUserMetrics);
    
    function Profile({user}) {
      const metrics = getUserMetrics(user);
      // ...
    }
    
    function TeamReport({users}) {
      for (let user in users) {
        const metrics = getUserMetrics(user);
        // ...
      }
      // ...
    }
    ```
    
    - **컴포넌트 안에서 호출하면**, 매번 새로운 캐시된 값을 반환하기때문에 무의미하다 !
        
        ```jsx
        // getWeekReport.js
        import {cache} from 'react';
        import {calculateWeekReport} from './report';
        
        /* 요런식으로 따로 빼서 다른 파일에서 import 해서 사용하세요 !*/
        export default cache(calculateWeekReport);
        ```
        
- 데이터의 스냅샷을 공유하기
    
    ```jsx
    import {cache} from 'react';
    import {fetchTemperature} from './api.js';
    
    const getTemperature = cache(async (city) => {
    	return await fetchTemperature(city);
    });
    
    // 이런식의 비동기 렌더링은 서버 컴포넌트에서만 가능
    async function AnimatedWeatherCard({city}) {
    	const temperature = await getTemperature(city);
    	// ...
    }
    
    // 이런식의 비동기 렌더링은 서버 컴포넌트에서만 가능
    async function MinimalWeatherCard({city}) {
    	const temperature = await getTemperature(city);
    	// ...
    }
    ```
    
- 데이터 preload 하기
    
    ```jsx
    const getUser = cache(async (id) => {
      return await db.user.query(id);
    }
    
    async function Profile({id}) {
      const user = await getUser(id);
      return (
        <section>
          <img src={user.profilePic} />
          <h2>{user.name}</h2>
        </section>
      );
    }
    
    function Page({id}) {
      // ✅ Good: start fetching the user data
      getUser(id);
      // ... some computational work
      return (
        <>
          <Profile id={id} />
        </>
      );
    }
    ```
    
    - page 에서 미리 `getUser` 를 호출해서 캐싱해두고, 
    Profile 이 렌더링될 때, 캐싱된 값으로 출력 !
    - `useMemo` , `memo` , `cache` 는 언제 사용할까요 ?
        
        `useMemo` : 일반적으로 client side 컴포넌트에서 사용합니다.
        
        `cache` : 일반적으로 서버 컴포넌트에서 사용합니다.
        
        `memo` : 컴포넌트가 리렌더링되는 것을 방지하려는 경우에 사용합니다.
