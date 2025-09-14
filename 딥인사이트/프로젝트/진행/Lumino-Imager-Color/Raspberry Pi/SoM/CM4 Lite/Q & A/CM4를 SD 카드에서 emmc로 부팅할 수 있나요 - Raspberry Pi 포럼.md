---
title: "CM4를 SD 카드에서 emmc로 부팅할 수 있나요? - Raspberry Pi 포럼"
source: "https://forums.raspberrypi.com/viewtopic.php?t=305506"
author:
published:
created: 2025-02-08
description: "라즈베리 파이 포럼에서 CM4를 SD 카드에서 emmc로 부팅하는 방법에 대한 논의.\
    \ 해당 스레드에서는 CM4의 부팅 순서, eMMC 비활성화, USB 부팅, EEPROM 설정 조정 등의\
    \ 다양한 방법이 제시됩니다."
tags:
  - "clippings"
---
[왕화](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=343005&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**게시물:** [7](https://forums.raspberrypi.com/search.php?author_id=343005&sr=posts&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**가입일:** 2020년 7월 25일 토요일 오전 1시 59분

### [SD 카드에서 emmc가 있는 CM4를 부팅할 수 있나요?](https://forums.raspberrypi.com/?t=305506#p1827784)

**질문:** CM4 (Compute Module 4) 에서 emmc를 비활성화하고 SD 카드에서 부팅하는 방법이 있는지 문의.

> 안녕하세요, 여러분:
> 저는 cm4와 cm4 io 보드를 구매했습니다. cm4에는 16G emmc가 있습니다. 제가 아는 대로, 저는 라즈베리를 emmc로 플래시하고 그것에서 부팅할 수 있습니다.
>
> emmc를 비활성화하고(손상시키지 않고) cm4를 SD 카드에서 부팅할 수 있는 방법이 있습니까? ![:연타:](https://forums.raspberrypi.com/images/smilies/icon_rolleyes.gif "눈을 굴리다")

---

[WH 하이트](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=13977&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**게시물:** [16955](https://forums.raspberrypi.com/search.php?author_id=13977&sr=posts&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**가입일:** 2012년 3월 9일 금요일 오후 7시 36분

**위치:** 캘리포니아주 발레호(미국)

### [Re: SD 카드에서 emmc가 있는 CM4가 부팅될 수 있나요?](https://forums.raspberrypi.com/?t=305506#p1827930)

[토요일 2월 27, 2021 오후 4:52](https://forums.raspberrypi.com/viewtopic.php?p=1827930&sid=fae29e9a4cb4b2b7d4e3de219e57babd#p1827930)

**답변:** CM4 모델에 따라 eMMC 플래시 용량이 다를 수 있음을 언급.

> [왕화는](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=343005&sid=fae29e9a4cb4b2b7d4e3de219e57babd) 이렇게 썼다: [↑](https://forums.raspberrypi.com/viewtopic.php?p=1827784&sid=fae29e9a4cb4b2b7d4e3de219e57babd#p1827784)
> 
> 토요일 2월 27, 2021 12:29 오후
> 
> cm4에는 16G emmc가 있습니다.
>
> 그것은 당신이 어떤 CM4 모델을 구매하느냐에 따라 달라집니다. eMMC 플래시는 없음에서 (기억이 맞다면) 32GB까지 다양합니다.

---

[왕화](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=343005&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**게시물:** [7](https://forums.raspberrypi.com/search.php?author_id=343005&sr=posts&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**가입일:** 2020년 7월 25일 토요일 오전 1시 59분

### [Re: SD 카드에서 emmc가 있는 CM4가 부팅될 수 있나요?](https://forums.raspberrypi.com/?t=305506#p1839343)

[토 3월 20, 2021 오전 9:57](https://forums.raspberrypi.com/viewtopic.php?p=1839343&sid=fae29e9a4cb4b2b7d4e3de219e57babd#p1839343)

**추가 질문:** 하드웨어 수정 (저항 제거, 신호 점퍼) 을 통해 SD 카드 부팅이 가능한지 문의.

> 알겠습니다. 감사합니다!
> 그러나 현재 CM4(16G EMMC 포함) 또는 IO 보드를 하드웨어 방식(예: 특정 저항 제거 또는 신호 점퍼)으로 약간 수정하여 SD 카드에서 부팅할 수 있습니까?

---


[타그롤](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=7855&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**게시물:** [13107](https://forums.raspberrypi.com/search.php?author_id=7855&sr=posts&sid=fae29e9a4cb4b2b7d4e3de219e57babd)

**가입일:** 2012년 1월 13일 금요일 오후 4시 41분

**위치:** 가장 어두운 서머싯, 영국

### [Re: SD 카드에서 emmc가 있는 CM4가 부팅될 수 있나요?](https://forums.raspberrypi.com/?t=305506#p1839406)

[토 3월 20, 2021 오후 1:20](https://forums.raspberrypi.com/viewtopic.php?p=1839406&sid=fae29e9a4cb4b2b7d4e3de219e57babd#p1839406)

**답변:**
*   SoC의 SD 핀이 EMMC로 직접 연결되어 있어 하드웨어 수정으로 SD 카드 부팅이 불가능함을 언급.
*   USB 카드 리더를 통한 SD 카드 부팅 가능성을 제시하며, EEPROM 설정 조정이 필요할 수 있음을 언급.

> [왕화는](https://forums.raspberrypi.com/memberlist.php?mode=viewprofile&u=343005&sid=fae29e9a4cb4b2b7d4e3de219e57babd) 이렇게 썼다: [↑](https://forums.raspberrypi.com/viewtopic.php?p=1839343&sid=fae29e9a4cb4b2b7d4e3de219e57babd#p1839343)
> 
> 토 3월 20, 2021 오전 9:57
> 
> 알겠습니다. 감사합니다!  
> 그러나 현재 CM4(16G EMMC 포함) 또는 IO 보드를 하드웨어 방식(예: 특정 저항 제거 또는 신호 점퍼)으로 약간 수정하여 SD 카드에서 부팅할 수 있습니까?
>
> [@timg236 이 위에서](https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=305506#p1827816) 말했듯이 , 아니요. SoC의 SD 핀은 CM4 커넥터가 아닌 EMMC로 라우팅됩니다.
>
> SD 카드를 USB 카드 리더에 삽입하여 부팅할 수 있지만 부팅 순서에 대한 EEPROM 설정을 조정해야 할 수도 있습니다.
>
> 지식, 기술, 경험은 가치가 있습니다. 누군가의 지식에서 이익을 기대한다면 그에 대한 비용을 지불해야 합니다.
>
> 제공된 모든 조언은 제 경험에 근거합니다. 저에게는 효과가 있었지만, 여러분에게는 효과가 없을 수 있습니다.
> 도움이 필요하세요? [https://github.com/thagrol/Guides](https://github.com/thagrol/Guides)

---
