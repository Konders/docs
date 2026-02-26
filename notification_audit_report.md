# Notification Audit Report

## Goal of review
Compare notification and WebSocket business scenarios in dev vs current, confirm behavioral equivalence or improvements, and document any changes, risks, and test coverage.

## Scope
- Compared: dev branch vs current working tree
- Areas: UnifiedGateway (WebSocket real-time), NotificationsProviderService (business events + channels)

## Business scenario map
Status legend:
- OK-equivalent = equivalent
- OK-improved = improved without changing business meaning
- WARN-different = behavior differs
- FAIL-broken = broken

### WebSocket and in-app real-time scenarios
| Scenario | Dev behavior | Current behavior | Status | Evidence |
| --- | --- | --- | --- | --- |
| WS-1 Connection and auth | Verify JWT, store socket in connectedClients map, set client.userId, register handlers | Verify JWT, set client.data.userId, join user:{id}, handlers via @SubscribeMessage | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#handleConnection, src/unified-gateway/unified-gateway.service.ts#handleConnection |
| WS-2 Request offer | No access check, save subscription in userOfferAssociations, respond to requesting socket | Check cashbox access, join offer:{id}, respond to user:{id} | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#requestOffer, src/unified-gateway/unified-gateway.service.ts#handleRequestOffer |
| WS-3 Offer updated | Iterate userOfferAssociations, fetch offer, emit to all user sockets | Use offer:{id} sockets to filter, fetch offer once, emit to user rooms | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#notifyOfferUpdate, src/unified-gateway/unified-gateway.service.ts#notifyOfferUpdate, src/unified-gateway/unified-gateway.service.spec.ts |
| WS-4 Request finop | Respond to requesting socket | Respond to user:{id} (all tabs) | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#requestFinop, src/unified-gateway/unified-gateway.service.ts#handleRequestFinop |
| WS-5 Cashier code request | Track order subscriptions in maps, respond to requesting socket | Join order:{id}, respond to user:{id} | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#cashierCodeRequest, src/unified-gateway/unified-gateway.service.ts#handleCashierCodeRequest |
| WS-6 Cashier code logout | Remove clientId from orderUserClientAssociations | Leave order:{id} | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#cashierCodeLogout, src/unified-gateway/unified-gateway.service.ts#handleCashierCodeLogout |
| WS-7 Cashier code refresh cron | Notify users subscribed to order (maps), then role-based users | Notify users in order:{id} room, then role-based users | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#handleCashierCodeUpdated, src/unified-gateway/unified-gateway.service.ts#handleCashierCodeUpdated, src/unified-gateway/unified-gateway.service.spec.ts |
| WS-8 Subscribe balance | No access check, record in userBalanceSubscriptions, respond to requesting socket | Check cashbox access, join balance:{id}, respond to user:{id} | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#subscribeBalance, src/unified-gateway/unified-gateway.service.ts#handleSubscribeBalance |
| WS-9 Unsubscribe balance | Remove cashbox from userBalanceSubscriptions | Leave balance:{id} | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#unsubscribeBalance, src/unified-gateway/unified-gateway.service.ts#handleUnsubscribeBalance |
| WS-10 Balance updated | Iterate userBalanceSubscriptions, compute per user, emit to all sockets | Use balance:{id} sockets to filter, compute per user, emit to user rooms | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#notifyBalanceUpdate, src/unified-gateway/unified-gateway.service.ts#notifyBalanceUpdate, src/unified-gateway/unified-gateway.service.spec.ts |
| WS-11 In-app notification (Telegram mini app) | sendNotification emits to all user sockets if connected | sendNotification emits to user:{id} if sockets exist | OK-equivalent | dev:src/unified-gateway/unified-gateway.service.ts#sendNotification, src/unified-gateway/unified-gateway.service.ts#sendNotification |
| WS-12 In-app admin notification | sendAdminNotification emits to all user sockets if connected | sendAdminNotification emits to user:{id} if sockets exist | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#sendAdminNotification, src/unified-gateway/unified-gateway.service.ts#sendAdminNotification |
| WS-13 Logout on access revoke | Emit logout to all sockets, no forced disconnect | Emit logout to user:{id} and disconnect sockets | OK-improved | dev:src/unified-gateway/unified-gateway.service.ts#logoutUserOnAccessRevoked, src/unified-gateway/unified-gateway.service.ts#logoutUserOnAccessRevoked |

### Business notification scenarios and channels
| Scenario | Dev behavior | Current behavior | Status | Evidence |
| --- | --- | --- | --- | --- |
| NP-1 Password change confirmation | Event routed via notificationConfig to ADMIN_PANEL_EMAIL | Same | OK-equivalent | src/notifications-provider/config/notification.config.ts#PASSWORD_CHANGE_CONFIRMATION |
| NP-2 Client created offer request (admin) | Send ADMIN_PANEL to exchange-related or access users | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#CLIENT_CREATED_OFFER_REQUEST_FOR_ADMIN |
| NP-3 Client created finop request (admin) | Send ADMIN_PANEL to exchange-related or access users | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#CLIENT_CREATED_FINOP_REQUEST_FOR_ADMIN |
| NP-4 Client submitted request (admin) | Validate order, send ADMIN_PANEL to exchange-related or access users | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#CLIENT_SUBMITTED_A_REQUEST_FOR_ADMIN |
| NP-5 Pending applications cron | Daily at 07:00, if total pending > 0 send ADMIN_PANEL | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#THERE_ARE_PENDINQ_APPLICATIONS |
| NP-6 Client created request (client) | If not banned, validate order, send TELEGRAM_MINI_APP | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#CLIENT_CREATED_A_REQUEST |
| NP-7 Admin changed status (client) | If not banned, send TELEGRAM_MINI_APP | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#ADMIN_CHANGED_STATUS_OF_REQUEST |
| NP-8 Client crypto transfer required | If not banned, send TELEGRAM_MINI_APP | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#CLIENT_CRYPTO_TRANSFER_REQUIRED |
| NP-9 Exchange payment reminder | Send ADMIN_PANEL to exchange-related or access users | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#handleAdminPaymentProcessing |
| NP-10 Export completed | If not banned, send ADMIN_PANEL | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#handleExportCompleted |
| NP-11 Export failed | If not banned, send ADMIN_PANEL | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#handleExportFailed |
| NP-12 Auto export completed | If not banned, send ADMIN_PANEL | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#handleAutoExportCompleted |
| NP-13 Auto export failed | If not banned, send ADMIN_PANEL | Same | OK-equivalent | src/notifications-provider/notifications-provider.service.ts#handleAutoExportFailed |

## Summary of changes
- Real-time delivery moved from in-memory maps to Socket.IO rooms with user:{id} fan-out.
- Responses and updates now arrive on all tabs for a user (when at least one tab subscribes).
- Added access checks for offer and balance subscriptions.
- Logout now disconnects sockets.
- Notifications provider logic and channel mapping are unchanged.

## What existed before (dev)
- Connection state and subscriptions stored in in-memory maps per user and per entity.
- requestOffer and subscribeBalance had no access control checks.
- Response events (offerRequested, finopRequested, cashierCodeResponse, balanceUpdate) were emitted only to the requesting socket.
- offerUpdated and balanceUpdated iterated user maps and emitted to all sockets of the user.
- Logout did not force socket disconnect.

## What exists now
- Rooms are used for both user fan-out (user:{id}) and entity subscription filtering (offer:{id}, balance:{id}, order:{id}).
- Access checks added for cashbox-bound subscriptions.
- Response events are emitted to user:{id} so all tabs stay synchronized.
- offerUpdated uses a single emit to user rooms, with one offer fetch.
- Logout emits and disconnects sockets in the user room.

## Detailed logic diff by scenario (with evidence)
- Request offer and subscribe balance: access check added before subscription and response fan-out. Evidence: src/unified-gateway/unified-gateway.service.ts#handleRequestOffer, src/unified-gateway/unified-gateway.service.ts#handleSubscribeBalance.
- Offer updates: now fetch once and emit to all user rooms in one call. Evidence: src/unified-gateway/unified-gateway.service.ts#notifyOfferUpdate.
- Finop responses: now sent to all tabs for the user, not only the requester. Evidence: src/unified-gateway/unified-gateway.service.ts#handleRequestFinop.
- Cashier code request: response is now user-level fan-out, while refresh cron behavior is preserved. Evidence: src/unified-gateway/unified-gateway.service.ts#handleCashierCodeRequest, src/unified-gateway/unified-gateway.service.ts#handleCashierCodeUpdated.
- Balance updates: entity room is only a filter; delivery is via user rooms. Evidence: src/unified-gateway/unified-gateway.service.ts#notifyBalanceUpdate.
- Admin notifications: now compute unreadCount once per send. Evidence: src/unified-gateway/unified-gateway.service.ts#sendAdminNotification.
- Notification provider events and channels unchanged. Evidence: src/notifications-provider/notifications-provider.service.ts, src/notifications-provider/config/notification.config.ts.

## Security
- Previous risks: no access checks for offer/balance subscriptions, no forced logout disconnects.
- Fixed now: access checks in requestOffer and subscribeBalance, forced disconnect on logout events.
- Remaining risks: possible duplicate delivery for cashierCodeRefreshCron if a user is both subscribed and role-based; no global deduplication. Behavior matches dev.

## Performance and optimization
- Previously: offerUpdated fetched offer per user iteration; admin notifications computed unreadCount per socket.
- Now: offerUpdated fetches once and emits to user rooms in one call; unreadCount computed once.
- Potential bottleneck: balanceUpdated computes balance per user sequentially; can be expensive with many subscribers.

## Test coverage
- UnifiedGateway has extensive unit coverage in src/unified-gateway/unified-gateway.service.spec.ts.
- NotificationsProviderService has unit tests in src/notifications-provider/notifications-provider.service.spec.ts.
- Gaps: no integration tests for multi-tab fan-out across rooms; no load tests for balanceUpdated.

## Final
- Merge decision: OK-improved (merge allowed)
- Action items:
  - Add integration test for multi-tab fan-out (offer and balance).
  - Consider batching or parallelizing balance calculations if subscriber count grows.
  - Optional: add explicit dedup for cashierCodeRefreshCron recipients.
