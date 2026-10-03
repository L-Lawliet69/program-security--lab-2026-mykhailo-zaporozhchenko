# Changelog — Gameplay Module

## [2026-10-03] [DOCS-001] Початкова специфікація ігрового рушія 1v1
- Описано кореневий агрегат `Match` та внутрішні сутності `PenaltyRound`, `ShotAttempt`.
- Зафіксовано обов'язкове бізнес-правило проведення повних 5 раундів (10 ударів) незалежно від дострокового лідерства за рахунком.
- Впроваджено політику `MustCompleteAllFiveShotsDPolicy` та сервіс `PenaltyResolutionService`.
- Додано захищений механізм жеребкування `CoinTossService` на базі CSPRNG.
