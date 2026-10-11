# Índices recomendados

Para PostgreSQL se recomienda considerar, como mínimo:

```text
users(email)
users(username)
artworks(author_id, created_at)
artworks(deleted, created_at)
artwork_comments(artwork_id, created_at)
artwork_ratings(artwork_id)
user_cards(user_id, card_id)
card_packs(user_id)
follows(follower_id)
follows(followed_id)
notifications(user_id, is_read, created_at)
artwork_reports(status, created_at)
comment_reports(status, created_at)
appeals(status, created_at)
appeals(report_target_type, report_id)  -- ya cubierto por el UNIQUE
email_verification_tokens(token_hash)   -- ya cubierto por el UNIQUE
email_change_tokens(token_hash)         -- ya cubierto por el UNIQUE
email_change_tokens(user_id, used_at)   -- invalidar tokens pendientes previos al solicitar un nuevo cambio
password_reset_tokens(token_hash)       -- ya cubierto por el UNIQUE
refresh_tokens(token_hash)              -- ya cubierto por el UNIQUE
```
