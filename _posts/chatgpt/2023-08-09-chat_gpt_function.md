---
layout: post
current: post
cover: assets/built/images/chatgpt/gpt-logo.jpg
navigation: True
class: post-template
author: sian
title: "[ChatGPT] 챗GPT function 기능 이용해보기"
comments: true
categories:
  - ChatGPT
tags:
  - ChatGPT, OpenAI
date: "2023-08-09 17:00:00"
---

## 서론

작년 말 출시하여 현재 1년도 안 된 사이에 엄청난 이용자가 생긴 AI 대화형 인공지능 ChatGPT(이하 챗GPT).

현재 GPT-4 모델 출시, 플러그인, 추가 기능 등등 많은 업데이트도 되고 있는 상황이다.

하지만 챗GPT의 주 목적은 사람과 자연스러운 대화를 나누는 것이기 때문에 간혹 모르는 주제의 내용임에도 잘못된 정보를 전달하는 등의 현상이 발생한다.

예를 들어 날씨를 알려달라고 한다거나, 비 유명인의 정보 등을 알려달라고 하면 그럴듯한 답변을 내놓지만 정보가 정확하지 않다.

이런 점 등을 OpenAI에서 추가로 제공하는 **function** 기능을 이용하여 보완을 할 수 있는데, 챗GPT를 이용하여 서비스를 제공하는 측에서
자신들의 커스텀 함수를 이용하여 챗GPT에게 추가로 자신들이 원하는 기능을 제공할 수 있게 만들 수 있다.

function 기능을 사용하여 정확한 날씨 정보를 응답하게 하거나, 자사 측 서비스를 활용하여 api를 사용하거나 메일을 보내는 등의 기능을 활용할 수 있다.

그래서 이 기능을 써보기 위해 OpenAI 설명서에 있는 function 기능을 한글로 나름 번역해서 테스트 해보았다.


### 환경

- gpt-3.5-turbo
- nextjs (테스트용)


## 1.	테스트용 앱 준비

기존에 챗GPT API를 테스트하기 위해 따로 nextjs로 만들어 둔 간단한 테스트 앱이 있는데, 이를 이용하여 function 테스트를 진행하였다.

![img](\assets\built\images\chatgpt\chatgpt_function_1.jpg)


## 2.	코드 입력

```js
// 날씨 정보를 알려주는 함수
function get_current_weather(location, unit = "섭씨") {
  const weather_info = {
    location: location,
    temperature: 30,
    unit: unit,
    forecast: ["맑음", "바람"],
  };
  return weather_info;
}

// 챗GPT api 요청
async function onSubmit(e) {
  e.preventDefault();

  // 메시지 작성중 체크
  if (msgCheck) {
    console.log("메세지 작성중입니다.");
    return;
  }

  // 기존 응답값 제거
  setResult("");

  // 챗gpt에 api 요청
  let thisForm = e.target;
  try {
    let messages = [{ role: "user", content: thisForm.input.value}];
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${process.env.NEXT_PUBLIC_OPENAI_API_KEY}`
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: messages,
        functions: [  // function 기능에 사용할 정보, 챗GPT가 프롬프트를 분석하여 function 기능이 필요한지 아닌지 분석하는데 사용된다.
          {
            name: "get_current_weather",
            description: "주어진 지역의 날씨 조회",
            parameters: {
              type: "object",
              properties: {
                location: {
                  type: "string",
                  description: "각 나라의 도시 혹은 특정 지역, 예를 들어 서울"
                },
                unit: {
                  type: "string",
                  enum: [
                    "섭씨",
                    "화씨"
                  ]
                }
              },
              required: ["location"]  // 사용자 정의 함수(get_current_weather 함수)에 필요한 필수 값. 챗gpt는 프롬프트를 분석하여 이 값을 리턴해준다.
            }
          }
        ],
        function_call: "auto" // function 기능을 자동으로 분석해서 사용할지 판단하도록 한다.
      }),
    });

    // 챗gpt api response
    const res = await response.json();
    console.log(res);

    // 응답문을 분석하여 챗gpt가 function 기능이 필요하다 판단되면
    // 응답문에 function_call이란 값을 리턴한다.
    // 이를 이용하여 function이 필요한지 아닌지 분기하여 function이 필요한 프롬프트라면
    // 기존에 미리 작성한 함수 등을 이용하여(현재는 위의 get_current_weather 함수) 추가 정보를 재차 챗gpt에 요청한다.
    // 그러면 챗gpt는 총 두번의 요청을 받은 뒤 사용자가 정의한 function대로 대답을 해주게 된다.
    const resMessage = res.choices[0].message;
    let result = "";
    if(resMessage.function_call) {  // function_call 값이 있는 지 확인
      const args = JSON.parse(resMessage.function_call.arguments);  // 챗gpt가 프롬프트를 분석해서 사용자 정의 함수에 필요한 필수 값을 리턴해준다.
      const functionRes = get_current_weather(args.location);       // 사용자 정의 함수로 챗gpt에 보낼 정보

      // 챗gpt에 보낼 값 재가공
      messages.push(resMessage);
      messages.push({
        role: "function",
        name: resMessage.function_call.name,
        content: JSON.stringify(functionRes)
      });

      // 2차 요청
      const secondResponse = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${process.env.NEXT_PUBLIC_OPENAI_API_KEY}`
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: messages,
        }),
      });

      // 챗gpt api 2차 response
      const secRes = await secondResponse.json();
      console.log(secRes);

      result = secRes.choices[0].message.content;
    } else {
      result = resMessage.content;
    }

    // 화면에 출력
    setResult(result);

  } catch (err) {
    // Consider implementing your own error handling logic here
    console.error(err);
    alert(err.message);
  } finally {
    setMsgCheck(false);
  }
}
```

아래에 링크해 둔 OpenAI의 가이드 글을 거의 그대로 가져와서 한글로 바꾸고 주석을 추가해놓은 javascript 코드이다.

위 코드의 주석에도 적어놓았지만, 간단하게 작동 방식을 설명하자면 아래와 같다.

1. 사용자가 챗GPT에게 API를 요청한다. 이 때, function을 사용하기 위하여 function 관련 정보를 같이 넘겨준다.
2. 챗GPT가 요청받은 function 관련 정보를 이용하여 프롬프트를 분석 후 function 기능이 필요한 문구인지 확인한다.
3. 챗GPT가 응답 시 function 기능이 필요한지 아닌지 여부를 포함하여 응답한다.
4. function 기능이 필요하지 않다면 그대로 응답문을 리턴한다.
5. function 기능이 필요하다면 사용자가 필요로 한 값을 리턴해주고 사용자는 이 값을 이용하여 자신의 함수를 동작시킨다.
6. 사용자 함수 동작 후 챗GPT에게 필요한 정보를 포함하여 2차 요청을 한다.
7. 챗GPT는 필요 정보를 받은 뒤, 사용자가 정의한 값을 이용하여 응답문을 리턴한다.


## 3.	결과 확인

function의 좋은 점은 사용자 함수를 사용한다는 점도 있지만, 챗GPT가 프롬프트를 분석하여 function이 필요하지 않은 문장이라면
평소처럼 일반적인 대답을 해준다는 점이다.

![img](\assets\built\images\chatgpt\chatgpt_function_2.jpg)

위 결과 화면처럼 function이 필요하지 않은 프롬프트라면 그냥 평소처럼 일반적인 대답을 해준다.

![img](\assets\built\images\chatgpt\chatgpt_function_3.jpg)

하지만 위 처럼 function이 필요한 구문이 들어온다면 챗GPT가 function이 필요한 기능이라는 1차 응답을 보내게 되고
이를 이용하여 사용자가 2차 요청을 보내면 2차 응답에서 최종적으로 사용자가 정의한 대로 응답을 받아볼 수 있게 된다.

옆에 같이 띄워놓은 콘솔에도 응답이 2번 온 모습을 확인할 수 있다.

![img](\assets\built\images\chatgpt\chatgpt_function_4.jpg)

참고로 function 기능을 사용하지 않으면 위처럼 날씨를 알 수 없다고 하거나, 잘못된 날씨 정보를 응답해준다.


## 추후 공부해야할 부분

이번엔 OpenAI쪽의 가이드 글을 그대로 사용해 본 것이라서 사용자가 2차 요청할 때 보낸 weather_info와 같은 사용자 값이
어떤 구조로 요청을 하면 되는 지에 대한 이해가 아직 낮은 상태이다.

OpenAI쪽에선 json 형식이면 된다고 하는 것 보니 json 형식만 지키면 key와 value 값도 알아서 분석하는 것이 아닌가 추측된다.


## 마치며

추후 사용자가 만든 API를 이용해본다던가, 다른 사람에게 메일을 보내본다던가 테스트 해 볼 여지가 아직 많이 남은 것 같다.


## 참조 사이트
[Chat GPT function 가이드 페이지]

[Chat GPT function 가이드 페이지]: https://platform.openai.com/docs/guides/gpt/function-calling	"Chat GPT function 가이드"