## Game > GameAnvil > CocosCreator 개발 가이드 > GameAnvil 커넥터 종료

## GameAnvil 커넥터 종료

게임 플레이 종료 전 Connector.CloseSoket() 함수를 호출해 연결을 종료하는 것이 좋습니다. 종료하지 않으면 서버에서 클라이언트의 종료를 인지하지 못할 수 있으며, 그럴 경우 불필요한 동작을 계속할 수 있습니다. Connector를 관리하는 컴포넌트를 게임 종료시에만 파괴 되도록 만들고 OnDestroy()에서 Connector.CloseSoket()를 호출하도록 만들어 놓는것도 좋은 방법입니다. 

