| Method | renderToNodeStream | renderToStaticNodeStream | renderToString | renderToStaticMarkup |
| ------- | ------- | ------- | ------- | ------- |
| **반환값** | Node.js 스트림 | Node.js 스트림 | HTML 문자열 | HTML 문자열 |
| **스트리밍 여부** | O | O | X | X |
| **Hydrate 가능 여부** | O | X | O | X |
| **특이사항** | Deprecated | 정적 페이지 생성 | 스트리밍 or 데이터 대기를 지원하지 않음 | 정적 페이지 생성 |
| **대안** | `renderToPipeableStream` | `renderToPipeableStream` | `renderToPipeableStream`, `renderToReadableStream`, `createRoot` | `renderToString` |
