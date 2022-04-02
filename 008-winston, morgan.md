## 0. 공부하게 된 계기
~이번 내용은 간단하다~  
단순히 개발 환경에서만 프로그램을 짜고, 개발을 끝마칠 거면 로그 파일에 대한 관리가 따로 필요 없을 것이다.  
하지만 개발을 마친 후에 서버를 배포할 것이고, 한 명이 그 콘솔을 24시간 내내 관찰하고 있을 수는 없다.  
따라서 로그를 따로 저장하는 것이 배포하기 전에 필수적이다.  
아직 배포 단계 전이지만 console.log 등을 하는 곳이 많아서 나중에 전부 고치기엔 상당한 시간이 걸릴 것 같아 개발 단계에서 미리 만들어줬다.  
node.js에서 로깅을 위해 자주 사용하는 모듈은 winston과 morgan이 있다.  
일반적으로는 두 모듈을 같이 사용하는 것 같다.  

## 1. 하는 일
##### winston
로그 파일 관리, 로그에 남길 메시지 관리, 로그 레벨 관리를 위한 모듈이다.  
공식문서에 따르면 요청을 위한 미들웨어를 제공하며 에러 로깅을 하는 모듈이라고 한다.  
요청과 응답으로부터 property를 선택하기 위해 화이트 리스트 방식을 채택하고 있다.  
사용을 위해 반드시 필요한 모듈은 'winston, winston-daily-rotate-file, app-root-path'이다.  
아래 코드와 이름만 봐도 간단한 사용법과 어떤 일을 하는지 알 수 있을 것이다.  

```
import winston from 'winston'
import winstonDaily from 'winston-daily-rotate-file'
import appRoot from 'app-root-path'


const logDir: string = 'logs';
const { combine, timestamp, printf } = winston.format;

const logFormat = printf(info => {
  return `${info.timestamp} ${info.level}: ${info.message}`;
});

const logger = winston.createLogger({
  format: combine(
    timestamp({
      format: 'YYYY-MM-DD HH:mm:ss',
    }),
    logFormat,
  ),
  transports: [
    new winstonDaily({
      level: 'info',
      datePattern: 'YYYY-MM-DD',
      dirname: `${appRoot}/${logDir}`,
      filename: `%DATE%.log`,
      maxFiles: 30,
      zippedArchive: true,
    }),
    new winstonDaily({
      level: 'error',
      datePattern: 'YYYY-MM-DD',
      dirname: `${appRoot}/${logDir}` + '/error',  // 파일 위치
      filename: `%DATE%.error.log`,  // 파일 이름
      maxFiles: 30,  // 30개까지의 파일만 생성, 그 이전 파일은 지움
      zippedArchive: true,
    }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),  // 로그에 색깔 가미
      winston.format.simple(),  // 로그를 간단히 고나리
    )
  }));
}

export default logger
```

위 모듈을 사용하고자 하는 위치에서 import logger ~를 한 뒤, logger.error, logger.info 등으로 사용할 수 있다.  

##### morgan
morgan도 http request에 대한 로그를 남겨주는 미들웨어이다.  
내가 작성한 코드에서는 morgan을 사용하지는 않았다.  
그래서 예시 코드는 없지만 사용법은 어렵지 않으니 공식문서 찾아보면 금방 익힐 것 같다.  

```
morgan(format, options)
```

위처럼 사용하면 morgan은 로그에 대한 형식과 옵션을 설정할 수 있다고 한다.
