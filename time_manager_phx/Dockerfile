
# === Frontend build stage ===
FROM node:20-alpine AS frontend-build
WORKDIR /frontend
COPY ../Frontend/package*.json ./
RUN npm install
COPY ../Frontend .
RUN npm run build

# === Backend build stage ===
FROM elixir:1.15-alpine AS build
RUN apk add --no-cache build-base git bash openssl postgresql-client
ENV MIX_ENV=prod
WORKDIR /app
RUN mix local.hex --force && mix local.rebar --force
COPY mix.exs mix.lock ./
COPY config config
RUN mix deps.get --only prod
RUN mix deps.compile
COPY lib lib
COPY priv priv
# Copy frontend build output to Phoenix static assets
COPY --from=frontend-build /frontend/dist ./priv/static
RUN mix compile
RUN mix release

# === Runtime stage ===
FROM elixir:1.15-alpine AS runtime
RUN apk add --no-cache bash openssl postgresql-client
WORKDIR /app
ENV MIX_ENV=prod
ENV SHELL=/bin/bash
COPY --from=build /app/_build/prod/rel/chrono_pulse ./
COPY entrypoint.sh ./
RUN chmod +x ./entrypoint.sh
ENTRYPOINT ["./entrypoint.sh"]
CMD ["bin/chrono_pulse", "start"]
