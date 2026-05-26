# Exemplo — Configurar RealtimeModule

## Arquivo: `src/infrastructure/realtime/realtime.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { RealtimeGateway } from './realtime.gateway';
import { RealtimeService } from './realtime.service';

@Module({
  imports: [
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.getOrThrow<string>('JWT_SECRET'),
      }),
    }),
  ],
  providers: [RealtimeGateway, RealtimeService],
  exports: [RealtimeService],
})
export class RealtimeModule {}
```

## Registro no AppModule

```typescript
import { RealtimeModule } from './infrastructure/realtime/realtime.module';

@Module({
  imports: [
    // ... outros módulos
    RealtimeModule,
  ],
})
export class AppModule {}
```

## Uso em um módulo de feature

```typescript
import { RealtimeModule } from '../infrastructure/realtime/realtime.module';

@Module({
  imports: [RealtimeModule],
  providers: [PedidoService],
})
export class PedidoModule {}
```

> O `RealtimeGateway` não é exportado — feature modules acessam apenas o `RealtimeService`.
