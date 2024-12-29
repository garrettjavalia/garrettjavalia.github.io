---
layout: post
title: "비즈니스 로직으로 리액트 컴포넌트 그리기"
date: 2021-11-28 04:01:00 +0900
categories: react
tags: react
published: true
# parent: "글"
---

# 개요

리액트의 장점은 그 선언적 구조에 있다. 현재 상태에 따라서 어떤 컴포넌트가 그려질지를 정해주는 방식으로 작성하는 코드는 버그가 발생할 확률을 낮춰준다. 그러나 때때로 리액트의 기본적인 코드 구조가 신경쓰고 싶지 않은 여러 상태를 부가적으로 관리하게 만들기도 한다. 오늘은 그 한 예시를 살펴보고 해결책도 만들어보자. 예시 코드는 생성형 AI를 사용해 만든 것이 있으므로 그 실행을 완전히 검증하지는 않았다. 해결책으로 제시하는 코드의 경우에도 그와 유사한 코드를 실제로 작성하고 동작도 검증했으나 이 문서에 적힌 그대로 실행을 보장하는 예제는 아니다.

# 문제 상황

아래는 일반적으로 쓰이는 리액트 다이얼로그 열기 코드이다.

```tsx
// 아래 코드는 copilt을 이용해서 생성했다. 개념을 보여주는 코드이므로 특별히 실행 검증을 하지는 않았다.
import React, { useState } from "react";
import Dialog from "@material-ui/core/Dialog";
import DialogActions from "@material-ui/core/DialogActions";
import DialogContent from "@material-ui/core/DialogContent";
import DialogContentText from "@material-ui/core/DialogContentText";
import DialogTitle from "@material-ui/core/DialogTitle";
import Button from "@material-ui/core/Button";

function SimpleDialog() {
  const [open, setOpen] = useState(false);

  const handleClickOpen = () => {
    setOpen(true);
  };

  const handleClose = () => {
    setOpen(false);
  };

  return (
    <div>
      <Button variant="outlined" color="primary" onClick={handleClickOpen}>
        Open dialog
      </Button>
      <Dialog open={open} onClose={handleClose}>
        <DialogTitle>{"Use Google's location service?"}</DialogTitle>
        <DialogContent>
          <DialogContentText>
            Let Google help apps determine location. This means sending
            anonymous location data to Google, even when no apps are running.
          </DialogContentText>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleClose} color="primary">
            Disagree
          </Button>
          <Button onClick={handleClose} color="primary" autoFocus>
            Agree
          </Button>
        </DialogActions>
      </Dialog>
    </div>
  );
}

export default SimpleDialog;
```

위 코드는 평범한 MUI 라이브러리 기반의 다이얼로그를 보여준다. 이 코드는 버튼으로 다이얼로그를 열고 닫는다. 보통 다이얼로그는 훨씬 복잡하고, 재사용성을 위해서 컴포넌트로 만들게된다. 그런 컴포넌트는 보통 아래와 같은 프로퍼티를 전달받는다.

```tsx
export interface CompnentStyleDialogProps {
  open: boolean;
  onClose: (result: any) => void; // 예시 코드이므로 any타입을 쓴다. 실제로는 비즈니스 로직에 따라 다를 것이다.
}
```

open변수로 다이얼로그가 보일지/숨을지를 결정하고, onClose에 넘겨주는 콜백함수는 보통 setOpen을 호출해서 다이얼로그를 닫을 책임을 질 것이다. 그 호출 예시는 이렇게 적을 수 있다.

```tsx
import React, { useState } from "react";
import ComponentStyleDialog, {
  CompnentStyleDialogProps,
} from "./ComponentStyleDialog";

function App() {
  const [dialogOpen, setDialogOpen] = useState(false);

  const handleDialogClose = (result: any) => {
    console.log("Dialog closed with result:", result);
    setDialogOpen(false);
  };

  const handleOpenDialog = () => {
    setDialogOpen(true);
  };

  return (
    <div>
      <Button variant="outlined" color="primary" onClick={handleOpenDialog}>
        Open Component Style Dialog
      </Button>
      <ComponentStyleDialog open={dialogOpen} onClose={handleDialogClose} />
    </div>
  );
}
```

만약 다이얼로그를 여러 개 띄워야하는 컴포넌트의 코드를 작성한다면 코드에는 비즈니스 로직 대신 다이얼로그를 띄우고 숨기거나 다이얼로그가 가질 상태값을 초기화하는 코드가 점점 많아진다. 이런 상황은 토스트(화면 한쪽에 나타났다가 사라지는 UI 요소)같은 UI를 가지는 컴포넌트를 작성할 때에 보일러플레이트 코드를 많이 작성해야 하는 것과도 비슷하다. 토스트 UI의 경우 전역 컨텍스트와 함께 동작하는 컴포넌트로 처리하는 등의 해결책을 생각할 수 있다. 다이얼로그도 유사하게 처리할 수 있다. 그러나 오늘 제시하는 코드는 전역 수준에서 컨텍스트나 컴포넌트를 추가할 필요 없이 비즈니스 로직에 집중한 코드이다.

# 해결책

## 아이디어

보통 모달 다이얼로그는 스스로 나타나고 스스로 사라지는 것이 이상적이다. 즉 외부에서 open/close를 직접 관리해줄 필요가 없어야한다. 그렇다면 우리가 상상할 수 있는 가장 편리한 사용법은 이런 식이라고 할 수 있다.

```ts
openDialog(TestDialog);
```

여기에 다이얼로그가 실행된 후에 그 결과값도 넘겨받고 싶다면 아마도 이런 방식을 원할 것이다.

```ts
openDialog(TestDialog).then((result) => {
  // 여기에 토스트 메시지를 보여주는 코드나 콘솔로그 코드가 올 수 있다.
});
```

그럼 이런 것을 만들기 위한 리액트 훅을 작성해보자.

처음에는 리액트 컴포넌트의 목록을 돌려주는 훅을 정의하는 것으로 시작하자.

```ts
export function useDialogView() {
  const [dialogs, setDialogs] = useState<React.ReactNode[]>([]);
  return [dialogs] as const;
}
```

이 훅을 사용하는 예시는 이렇게 쓸 수 있다.

```tsx
export function App() {
  const [dialogs] = useDialog();
  return <>{...dialogs}</>;
}
```

## 가장 단순한 버전

이제 useDialog에 어떤 방식으로 임의의 리액트 컴포넌트를 추가할 수 있게 해야한다. 이렇게 작성해보자.

```tsx
export function useDialogView() {
  const [dialogs, setDialogs] = useState<React.ReactNode[]>([]);
  const openDialog(dialogFunc: () => React.ReactNode) => {
    setDialogs(oldDialogs => {
      return [...oldDialogs, createElement(dialogFunc)]
    })
  }
  return [dialogs, openDialog] as const;
}
```

이렇게 만든 코드는 아래처럼 쓸 수 있다.

```tsx
function App() {
  const [dialogs, openDialog] = useDialogView();

  const handleOpenDialog = () => {
    openDialog(TestDialog);
  };

  return (
    <div>
      <button onClick={handleOpenDialog}>Open Dialog</button>
      {...dialogs}
    </div>
  );
}
```

## 프롭 넘기기의 문제

만약 openDialog함수에 다이얼로그의 props를 넘겨야 한다면 그 기능을 구현하고 사용하는 코드는 이런 모습이 될 것이다.

```jsx
// 단순함을 위해서 type정보를 생략했다.
export function useDialogView() {
  const [dialogs, setDialogs] = useState<React.ReactNode[]>([]);
  const openDialog(dialogFunc: () => React.ReactNode, args:any[]) => {
    setDialogs(oldDialogs => {
      return [...oldDialogs, createElement(dialogFunc, any)]
    })
  }
  return [dialogs, openDialog] as const;
}
```

```tsx
openDialog(TestDialog, { p1: string, p2: number });
```

## 호출부가 JSX표기를 쓸 수 있게 하기

그런데 이 코드는 다이얼로그를 여는 코드가 흔히 아는 jsx표기법을 벗어난다는 문제가 있다. 해결을 위해서 openDialog를 조금 비틀어보자.

```ts
export function useDialogView() {
  const [dialogs, setDialogs] = useState<React.ReactNode[]>([]);

  function openDialog(componentGenerator: () => ReactNode) {
    setDialogs((oldDialogs) => {
      return [...oldDialogs, componentGenerator()];
    });
  }

  return [dialogIdPairs.map((dialog) => dialog.dialog), openDialog] as const;
}
```

이렇게 구성한 useDialogView는 이제 아래와 같이 사용한다. openDialog의 호출부가 리액트 기준으로 자연스러워졌다.

```tsx
function App() {
  const [dialogs, openDialog] = useDialogView();

  const handleOpenDialog = () => {
    openDialog(() => <TestDialog p1="example" p2={42} />);
  };

  return (
    <div>
      <button onClick={handleOpenDialog}>Open Dialog</button>
      {...dialogs}
    </div>
  );
}
```

## 닫기 콜백 처리

이제 우리가 만든 다이얼로그가 종료될 때 콜백 함수가 호출되게 하고싶다. 이렇게 하기위해서 우리가 openDialog로 열 수 있는 다이얼로그 컴포넌트가 공통으로 가지는 프롭을 정의하자.

```tsx
export interface DialogViewDialogProps {
  onClose: (closePayload: ClosePayloadType) => void;
  onDispose: () => void;
}

type ExtractPayloadAndClose<T> = T extends DialogViewDialogProps<
  infer PayloadType,
  infer ClosePayloadType
>
  ? [PayloadType, ClosePayloadType]
  : never;
```

이제 닫기를 처리해야 하므로 우리의 코드는 추가되는 모든 다이얼로그 컴포넌트에 고유아이디를 자동으로 부여하고 관리해야 한다. 또한 콜백은 비동기로 호출되어야 하므로 Promise를 사용하도록 한다. 그 코드는 아래와 같이 완성된다.

```tsx
export function useDialogView() {
  const [nextDialogId, setNextDialogId] = useState(0);
  const [dialogIdPairs, setDialogIdPairs] = useState<DialogWithId[]>([]);

  function openDialog<T extends DialogViewDialogProps<any, any>>(
    componentGenerator: (
      key: Key,
      onClose: (payload: ExtractPayloadAndClose<T>[1]) => void,
      onDispose: () => void
    ) => ReactNode,
    manualDispose?: boolean
  ) {
    return new Promise<ExtractPayloadAndClose<T>[1]>((resolve, reject) => {
      const dialogId = nextDialogId;
      setNextDialogId((value) => value + 1);

      const disposer = () => {
        setDialogIdPairs((value) => {
          const searchedDialog = binarySearch(value, dialogId, (a, b) => {
            if (a.id < b) return -1;
            if (a.id > b) return 1;
            return 0;
          });
          if (!searchedDialog) return value;
          return value
            .slice(0, searchedDialog.index)
            .concat(value.slice(searchedDialog.index + 1));
        });
        reject("disposed");
      };

      const closer = (payload: ExtractPayloadAndClose<T>[1]) => {
        resolve(payload);
        if (!manualDispose) {
          setTimeout(() => {
            disposer();
          }, 1000);
        }
      };

      setDialogIdPairs([
        ...dialogIdPairs,
        {
          id: dialogId,
          dialog: componentGenerator(dialogId, closer, disposer),
        },
      ]);
    });
  }

  return [dialogIdPairs.map((dialog) => dialog.dialog), openDialog] as const;
}
```

위 코드에서는 배열에 들어가는 다이얼로그 컴포넌트의 키값이 늘 오름차순으로 놓인다는 점을 생각해서 binarySearch를 사용해 동작하도록 구성했다. 그러나 실제 사용 예시에서는 이진탐색이 필요할 정도로 많은 다이얼로그를 사용하는 경우는 없을 것이다.

한편 위의 코드에서는 생성하는 다이얼로그의 key를 componentGenerator함수에 넘기고 있으므로 key를 지정하는 작업을 다이얼로그가 수행해야 한다는 문제가 있다. 이것은 <Fragment>요소를 직접 사용해서 key를 미리 넣어주는 방식으로 하면 문제가 없으나 리액트 컴포넌트를 Fragment로 감싸는 동작은 직계 자식 컴포넌트들의 타입을 따지는 부모 컴포넌트를 사용해야 할 경우 문제가 되므로 일부러 피했다. (물론 다이얼로그 컴포넌트를 감싸는 코드에서는 그런 동작을 할 여지가 적지만 그런 경우가 있다는 정도로 기억하자.)

또한 dispose와 close라는 두 종류의 작업을 제시하고 있는데, close는 다이얼로그가 화면에서 시각적으로 사라지는 상황을 처리하며 dispose는 다이얼로그가 렌더 트리에서 제거되는 상황을 처리한다. 이것은 다이얼로그가 사라지는 애니메이션이 있는 경우에 그 처리를 어느정도 쉽게 하기 위한 배려이다. 그래서 dispose작업은 기본적으로 자동으로 실행되도록 만들었다. 또한 useDialogView내부의 리액트 상태 설정 함수가 모두 값을 넘겨받는 콜백 형태인 것을 볼 수 있는데 이것은 다이얼로그 여러 개가 생성된 상황에서 올바른 동작을 보장해준다.

최종적인 사용 예시는 아래와 같다.

```tsx
function App() {
  const [dialogs, openDialog] = useDialogView();
  const onOpenDialog = () => {
    openDialog((key, onClose, onDispose) => {
      return (
        <TestDialog
          key={key}
          onClose={onClose}
          onDispose={onDispose}
          initialData={undefined}
        ></TestDialog>
      );
    }).then((payload) => {
      console.log(payload, "closed");
    });
  };
  return (
    <>
      {dialogs}
      <Button onClick={onOpenDialog} />
    </>
  );
}
```

위에서 제시한 useDialogView 함수는 다이얼로그를 부모 컴포넌트가 강제로 닫는 로직을 제공하지 않는다. 그러나 조금만 응용하면 useDialogView 훅을 호출할 때 공용 disposer를 같이 돌려주게 하거나 위의 openDialog에 넘겨주는 component generator함수에 해당 다이얼로그 전용의 disposer를 넘겨주도록 구현할 수 있을 것이다.
