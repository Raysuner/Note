思路：
1）用tab的个数乘以为每个tab设定的最大宽度来设置tab容器宽度，方便平移。
2）使用move函数计算出需要平移的距离
- 利用最后一个的tab的offsetLeft和自身宽度来判断是否宽于整个tabbar宽度，如果没有的话，整个tab容器不需要偏移。
- 处理好因为偏移导致第一个tab和最后一个tab因为偏移导致和两边边界有视野范围内有不合理的距离。

```jsx
import React, { useEffect, useRef, useState } from 'react';
import './index.css';

const TAB_COUNT = 20;
const MAX_ITEM_WIDTH = 300;
const TAB_BAR_WIDTH = 1200;
const MOVE_STEP_DISTANCE = TAB_BAR_WIDTH / 2;

function ScrollTab(props) {
  const tabListRef = useRef(null);

  const [activeIndex, setActiveIndex] = useState(0);
  const [scrollOffset, setScrollOffset] = useState(0); // 滚动偏移量，使激活的 Tab 居中
  const [showLeftArrow, setShowLeftArrow] = useState(false);
  const [showRightArrow, setShowRightArrow] = useState(false);

  const getTab = (idx) => {
    return tabListRef.current.childNodes[idx];
  };

  const move = (distance) => {
    let offset = distance,
      leftShow = true,
      rightShow = true;

    const firstTab = getTab(0);
    const lastTab = getTab(TAB_COUNT - 1);

    if (lastTab.offsetLeft + lastTab.offsetWidth < TAB_BAR_WIDTH) {
      offset = 0;
      leftShow = rightShow = false;
    } else {
      if (firstTab.offsetLeft + distance > 0) {
        offset = 0;
        leftShow = false;
      }
      if (lastTab.offsetLeft + lastTab.offsetWidth + distance < TAB_BAR_WIDTH) {
        offset = TAB_BAR_WIDTH - lastTab.offsetLeft - lastTab.offsetWidth;
        rightShow = false;
      }
    }
    setShowLeftArrow(leftShow);
    setShowRightArrow(rightShow);
    setScrollOffset(offset);
  };

  useEffect(() => {
    const activeTab = getTab(activeIndex);
    let distance = MOVE_STEP_DISTANCE - activeTab.offsetLeft;
    move(distance);
  }, [activeIndex]);

  return (
    <div className="tab-container">
      <div
        className="tab-list"
        ref={tabListRef}
        style={{
          width: `${TAB_COUNT * MAX_ITEM_WIDTH}px`,
          transform: `translateX(${scrollOffset}px)`
        }}
      >
        {[...new Array(TAB_COUNT)].map((_, i) => (
          <div
            key={i}
            className={`tab-item ${i === activeIndex ? 'active' : ''}`}
            onClick={() => {
              if (activeIndex === i) return;
              setActiveIndex(i);
            }}
          >
            Tab {i + 1}
          </div>
        ))}
      </div>
      {showLeftArrow && (
        <div
          className="left-arrow"
          onClick={() => move(scrollOffset + TAB_BAR_WIDTH)}
        >
          left
        </div>
      )}
      {showRightArrow && (
        <div
          className="right-arrow"
          onClick={() => move(scrollOffset - TAB_BAR_WIDTH)}
        >
          right
        </div>
      )}
    </div>
  );
}

export default ScrollTab;
```