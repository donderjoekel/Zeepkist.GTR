# GTR

### Note
Hosting GTR **costs money**. You need to host the docker images on a server somewhere (either your own or a rented one) and you need to pay for file storage.

# Requirements
Hosting GTR comes with a couple of requirements

1. A server that can run the containers
   - I rented a server from [Hetzner](https://hetzner.com), specifically the CX32 model
2. An [OpenObserve](https://github.com/openobserve/openobserve) instance for logging (which you can host yourself)
3. A storage provider, more specifically [Wasabi](https://wasabi.com), for storing workshop data

Once you have all that you can move on to the next step

# Installation

- Copy the contents of the `Compose` section into a file named `docker-compose.yml`
- Copy the contents of the `Env File` section into a file named `.env`
   - Make sure this file is placed besides the `docker-compose.yml` file for it to be automatically loaded
- Fill in all the fields in your newly created `.env` file
- Run `docker compose up -d` to start your instance

Once your instance is up and running you can use the `SQL Dump` section to create your database tables.

After all this you should be good to go!

### Compose

```yaml
services:

  backend:
    image: "tnrd/zeepkist.gtr.backend:latest"
    hostname: "backend"
    container_name: "backend"
    environment:
      - "DATABASE__CONNECTIONSTRING=Host=${DB_HOST};Port=${DB_PORT};Database=${DB_DATABASE};Username=${DB_USERNAME};Password=${DB_PASSWORD}"
      - "JOBS__ENABLEWORKSHOP=false" # Keep this to FALSE, this is for the workshop container
      - "JWT__AUDIENCE=${JWT_AUDIENCE}"
      - "JWT__ISSUER=$JWT_ISSUER}"
      - "JWT__TOKEN=${JWT_TOKEN}"
      - "LOGGER__URL=${OPENOBSERVE_URL}"
      - "LOGGER__LOGIN=${OPENOBSERVE_LOGIN}"
      - "LOGGER__TOKEN=${OPENOBSERVE_TOKEN"
      - "LOGGER__STREAM=${OPENOBSERVE_STREAM}"
      - "WASABISTORAGE__SERVICEURL=${WASABI_URL}"
      - "WASABISTORAGE__BUCKET=${WASABI_BUCKET}"
      - "WASABISTORAGE__ACCESSKEYID=${WASABI_ACCESSKEY}"
      - "WASABISTORAGE__SECRETACCESSKEY=${WASABI_SECRETKEY}"
      - "STEAM__APIKEY=${STEAM_API_KEY}"
      - "WORKSHOP__STEAMCMDPATH=${STEAMCMD_PATH}"
      - "WORKSHOP__MOUNTPATH=${WORKSHOP_PATH}"

  workshop:
    hostname: "workshop"
    image: "tnrd/zeepkist.gtr.backend:latest"
    container_name: "workshop"
    environment:
      - "DATABASE__CONNECTIONSTRING=Host=${DB_HOST};Port=${DB_PORT};Database=${DB_DATABASE};Username=${DB_USERNAME};Password=${DB_PASSWORD}"
      - "JOBS__ENABLEWORKSHOP=true" # Keep this to TRUE, this will process the workshop
      - "JWT__AUDIENCE=${JWT_AUDIENCE}"
      - "JWT__ISSUER=$JWT_ISSUER}"
      - "JWT__TOKEN=${JWT_TOKEN}"
      - "LOGGER__URL=${OPENOBSERVE_URL}"
      - "LOGGER__LOGIN=${OPENOBSERVE_LOGIN}"
      - "LOGGER__TOKEN=${OPENOBSERVE_TOKEN"
      - "LOGGER__STREAM=${OPENOBSERVE_STREAM}"
      - "WASABISTORAGE__SERVICEURL=${WASABI_URL}"
      - "WASABISTORAGE__BUCKET=${WASABI_BUCKET}"
      - "WASABISTORAGE__ACCESSKEYID=${WASABI_ACCESSKEY}"
      - "WASABISTORAGE__SECRETACCESSKEY=${WASABI_SECRETKEY}"
      - "STEAM__APIKEY=${STEAM_API_KEY}"
      - "WORKSHOP__STEAMCMDPATH=${STEAMCMD_PATH}"
      - "WORKSHOP__MOUNTPATH=${WORKSHOP_PATH}"

  graphql:
    # This is a custom "fork" of the postgraphile image with some added plugins 
    image: "tnrd/postgraphile:latest"
    hostname: "postgraphile"
    container_name: "graphql"
    command:
      - "--connection"
      - "postgres://${DB_USERNAME}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_DATABASE}"
      - "--schema"
      - "public"
      - "--watch"
      - "--enhance-graphiql"
      - "--disable-default-mutations"
      - "-q"
      - "/"
      - "--append-plugins"
      - "postgraphile-plugin-connection-filter-relations"
      - "--cors"

  postgres:
    hostname: "postgres"
    image: "postgres:13"
    container_name: "postgres"
    environment:
      - "POSTGRES_DB=${DB_DATABASE}"
      - "POSTGRES_USER=${DB_USERNAME}"
      - "POSTGRES_PASSWORD=${DB_PASSWORD}"
      - "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/13/bin"
      - "GOSU_VERSION=1.17"
      - "LANG=en_US.utf8"
      - "PG_MAJOR=13"
      - "PG_VERSION=13.16-1.pgdg120+1"
      - "PGDATA=/var/lib/postgresql/data"
    volumes:
      - "${DB_PATH}:/var/lib/postgresql/data"
```

### Env file
```bash
# Database information
DB_HOST=
DB_PORT=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
DB_PATH=/opt/db

# JWT information for handling authentication
JWT_AUDIENCE=
JWT_ISSUER=
JWT_TOKEN=

# Information for your OpenObserve instance
OPENOBSERVE_URL=
OPENOBSERVE_LOGIN=
OPENOBSERVE_TOKEN=
OPENOBSERVE_STREAM=

# Information about your Wasabi storage, get yours here: https://wasabi.com 
WASABI_URL=
WASABI_BUCKET=
WASABI_ACCESSKEY=
WASABI_SECRETKEY=

# Your steam web api key, get yours here: https://steamcommunity.com/dev/apikey
STEAM_API_KEY=

# The docker container automatically installs steamcmd, so unless you're changing this yourself you should leave this as is
STEAMCMD_PATH=steamcmd.sh

# The path where you want the workshop data to be stored
WORKSHOP_PATH=/opt/workshop
```

### SQL Dump
```sql
--
-- PostgreSQL database dump
--

-- Dumped from database version 13.16 (Debian 13.16-1.pgdg120+1)
-- Dumped by pg_dump version 13.16 (Debian 13.16-1.pgdg120+1)

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- Name: public; Type: SCHEMA; Schema: -; Owner: -
--

CREATE SCHEMA public;


--
-- Name: SCHEMA public; Type: COMMENT; Schema: -; Owner: -
--

COMMENT ON SCHEMA public IS 'standard public schema';


SET default_tablespace = '';

SET default_table_access_method = heap;

--
-- Name: auth; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.auth (
    id integer NOT NULL,
    "user" integer,
    access_token character varying(255),
    access_token_expiry character varying(255),
    refresh_token character varying(255),
    refresh_token_expiry character varying(255),
    type integer,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: TABLE auth; Type: COMMENT; Schema: public; Owner: -
--

COMMENT ON TABLE public.auth IS '@omit';


--
-- Name: auth_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.auth ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.auth_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: favorites; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.favorites (
    id integer NOT NULL,
    "user" integer NOT NULL,
    level text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: favorites_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.favorites ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.favorites_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: level_points; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.level_points (
    id integer NOT NULL,
    level text NOT NULL,
    points integer NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: level_points_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.level_points ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.level_points_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: levels; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.levels (
    id integer NOT NULL,
    name text NOT NULL,
    image_url text NOT NULL,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL,
    workshop_id numeric NOT NULL,
    author_id numeric NOT NULL,
    file_hash text NOT NULL,
    file_url text NOT NULL,
    file_author text NOT NULL,
    file_uid text NOT NULL,
    replaced_by integer,
    deleted boolean NOT NULL,
    metadata_id integer NOT NULL
);


--
-- Name: levels_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.levels ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.levels_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: media; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.media (
    id integer NOT NULL,
    record integer NOT NULL,
    ghost_url text NOT NULL,
    screenshot_url text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: media_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.media ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.media_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: metadata; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.metadata (
    hash text NOT NULL,
    valid boolean NOT NULL,
    checkpoints integer NOT NULL,
    blocks text NOT NULL,
    validation real NOT NULL,
    gold real NOT NULL,
    silver real NOT NULL,
    bronze real NOT NULL,
    ground integer NOT NULL,
    skybox integer NOT NULL,
    id integer NOT NULL
);


--
-- Name: metadata_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.metadata ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.metadata_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: personal_bests; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.personal_bests (
    id integer NOT NULL,
    record integer NOT NULL,
    "user" integer NOT NULL,
    period_start timestamp with time zone,
    period_end timestamp with time zone,
    level text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: personal_bests_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.personal_bests ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.personal_bests_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: player_points; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.player_points (
    id integer NOT NULL,
    "user" integer NOT NULL,
    points integer NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL,
    rank integer DEFAULT 0 NOT NULL,
    world_records integer DEFAULT 0
);


--
-- Name: player_points_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.player_points ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.player_points_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: records; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.records (
    id integer NOT NULL,
    "user" integer NOT NULL,
    "time" real NOT NULL,
    splits text,
    game_version character varying(255) NOT NULL,
    is_valid boolean NOT NULL,
    level text NOT NULL,
    mod_version text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: records_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.records ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.records_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: requests; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.requests (
    id integer NOT NULL,
    workshop_id numeric NOT NULL,
    uid text,
    hash text,
    date_created timestamp with time zone NOT NULL
);


--
-- Name: requests_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.requests ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.requests_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: sampled_auth; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_auth AS
 SELECT auth.id,
    auth."user",
    auth.access_token,
    auth.access_token_expiry,
    auth.refresh_token,
    auth.refresh_token_expiry,
    auth.type,
    auth.date_created,
    auth.date_updated
   FROM public.auth TABLESAMPLE bernoulli (1);


--
-- Name: sampled_favorites; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_favorites AS
 SELECT favorites.id,
    favorites."user",
    favorites.level,
    favorites.date_created,
    favorites.date_updated
   FROM public.favorites TABLESAMPLE bernoulli (1);


--
-- Name: sampled_level_points; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_level_points AS
 SELECT level_points.id,
    level_points.level,
    level_points.points,
    level_points.date_created,
    level_points.date_updated
   FROM public.level_points TABLESAMPLE bernoulli (1);


--
-- Name: sampled_levels; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_levels AS
 SELECT levels.id,
    levels.name,
    levels.image_url,
    levels.created_at,
    levels.updated_at,
    levels.workshop_id,
    levels.author_id,
    levels.file_hash,
    levels.file_url,
    levels.file_author,
    levels.file_uid,
    levels.replaced_by,
    levels.deleted,
    levels.metadata_id
   FROM public.levels TABLESAMPLE bernoulli (1);


--
-- Name: sampled_media; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_media AS
 SELECT media.id,
    media.record,
    media.ghost_url,
    media.screenshot_url,
    media.date_created,
    media.date_updated
   FROM public.media TABLESAMPLE bernoulli (1);


--
-- Name: sampled_metadata; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_metadata AS
 SELECT metadata.hash,
    metadata.valid,
    metadata.checkpoints,
    metadata.blocks,
    metadata.validation,
    metadata.gold,
    metadata.silver,
    metadata.bronze,
    metadata.ground,
    metadata.skybox,
    metadata.id
   FROM public.metadata TABLESAMPLE bernoulli (1);


--
-- Name: sampled_personal_bests; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_personal_bests AS
 SELECT personal_bests.id,
    personal_bests.record,
    personal_bests."user",
    personal_bests.period_start,
    personal_bests.period_end,
    personal_bests.level,
    personal_bests.date_created,
    personal_bests.date_updated
   FROM public.personal_bests TABLESAMPLE bernoulli (1);


--
-- Name: sampled_player_points; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_player_points AS
 SELECT player_points.id,
    player_points."user",
    player_points.points,
    player_points.date_created,
    player_points.date_updated,
    player_points.rank,
    player_points.world_records
   FROM public.player_points TABLESAMPLE bernoulli (1);


--
-- Name: sampled_records; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_records AS
 SELECT records.id,
    records."user",
    records."time",
    records.splits,
    records.game_version,
    records.is_valid,
    records.level,
    records.mod_version,
    records.date_created,
    records.date_updated
   FROM public.records TABLESAMPLE bernoulli (1);


--
-- Name: sampled_requests; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_requests AS
 SELECT requests.id,
    requests.workshop_id,
    requests.uid,
    requests.hash,
    requests.date_created
   FROM public.requests TABLESAMPLE bernoulli (1);


--
-- Name: sampled_spatial_ref_sys; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_spatial_ref_sys AS
 SELECT spatial_ref_sys.srid,
    spatial_ref_sys.auth_name,
    spatial_ref_sys.auth_srid,
    spatial_ref_sys.srtext,
    spatial_ref_sys.proj4text
   FROM public.spatial_ref_sys TABLESAMPLE bernoulli (1);


--
-- Name: upvotes; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.upvotes (
    id integer NOT NULL,
    "user" integer NOT NULL,
    level text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: sampled_upvotes; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_upvotes AS
 SELECT upvotes.id,
    upvotes."user",
    upvotes.level,
    upvotes.date_created,
    upvotes.date_updated
   FROM public.upvotes TABLESAMPLE bernoulli (1);


--
-- Name: users; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.users (
    id integer NOT NULL,
    steam_id character varying(255),
    steam_name character varying(255),
    discord_id character varying(255),
    banned boolean DEFAULT false NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: sampled_users; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_users AS
 SELECT users.id,
    users.steam_id,
    users.steam_name,
    users.discord_id,
    users.banned,
    users.date_created,
    users.date_updated
   FROM public.users TABLESAMPLE bernoulli (1);


--
-- Name: versions; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.versions (
    id integer NOT NULL,
    minimum text,
    latest text,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: sampled_versions; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_versions AS
 SELECT versions.id,
    versions.minimum,
    versions.latest,
    versions.date_created,
    versions.date_updated
   FROM public.versions TABLESAMPLE bernoulli (1);


--
-- Name: votes; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.votes (
    id integer NOT NULL,
    "user" integer NOT NULL,
    score integer NOT NULL,
    level text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: sampled_votes; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_votes AS
 SELECT votes.id,
    votes."user",
    votes.score,
    votes.level,
    votes.date_created,
    votes.date_updated
   FROM public.votes TABLESAMPLE bernoulli (1);


--
-- Name: world_records; Type: TABLE; Schema: public; Owner: -
--

CREATE TABLE public.world_records (
    id integer NOT NULL,
    record integer NOT NULL,
    "user" integer NOT NULL,
    period_start timestamp with time zone,
    period_end timestamp with time zone,
    level text NOT NULL,
    date_created timestamp with time zone DEFAULT now() NOT NULL,
    date_updated timestamp with time zone DEFAULT now() NOT NULL
);


--
-- Name: sampled_world_records; Type: VIEW; Schema: public; Owner: -
--

CREATE VIEW public.sampled_world_records AS
 SELECT world_records.id,
    world_records.record,
    world_records."user",
    world_records.period_start,
    world_records.period_end,
    world_records.level,
    world_records.date_created,
    world_records.date_updated
   FROM public.world_records TABLESAMPLE bernoulli (1);


--
-- Name: upvotes_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.upvotes ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.upvotes_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: users_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.users ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.users_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: versions_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.versions ALTER COLUMN id ADD GENERATED ALWAYS AS IDENTITY (
    SEQUENCE NAME public.versions_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: votes_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.votes ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.votes_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: world_records_id_seq; Type: SEQUENCE; Schema: public; Owner: -
--

ALTER TABLE public.world_records ALTER COLUMN id ADD GENERATED BY DEFAULT AS IDENTITY (
    SEQUENCE NAME public.world_records_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1
);


--
-- Name: auth auth_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.auth
    ADD CONSTRAINT auth_pkey PRIMARY KEY (id);


--
-- Name: favorites favorites_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.favorites
    ADD CONSTRAINT favorites_pkey PRIMARY KEY (id);


--
-- Name: level_points level_points_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.level_points
    ADD CONSTRAINT level_points_pkey PRIMARY KEY (id);


--
-- Name: levels levels_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.levels
    ADD CONSTRAINT levels_pkey PRIMARY KEY (id);


--
-- Name: media media_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.media
    ADD CONSTRAINT media_pkey PRIMARY KEY (id);


--
-- Name: metadata metadata_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.metadata
    ADD CONSTRAINT metadata_pkey PRIMARY KEY (id);


--
-- Name: personal_bests personal_bests_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.personal_bests
    ADD CONSTRAINT personal_bests_pkey PRIMARY KEY (id);


--
-- Name: player_points player_points_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.player_points
    ADD CONSTRAINT player_points_pkey PRIMARY KEY (id);


--
-- Name: records records_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.records
    ADD CONSTRAINT records_pkey PRIMARY KEY (id);


--
-- Name: requests requests_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.requests
    ADD CONSTRAINT requests_pkey PRIMARY KEY (id);


--
-- Name: upvotes upvotes_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.upvotes
    ADD CONSTRAINT upvotes_pkey PRIMARY KEY (id);


--
-- Name: users users_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.users
    ADD CONSTRAINT users_pkey PRIMARY KEY (id);


--
-- Name: versions versions_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.versions
    ADD CONSTRAINT versions_pkey PRIMARY KEY (id);


--
-- Name: votes votes_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.votes
    ADD CONSTRAINT votes_pkey PRIMARY KEY (id);


--
-- Name: world_records world_records_pkey; Type: CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.world_records
    ADD CONSTRAINT world_records_pkey PRIMARY KEY (id);


--
-- Name: IX_auth_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_auth_user" ON public.auth USING btree ("user");


--
-- Name: IX_favorites_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_favorites_user" ON public.favorites USING btree ("user");


--
-- Name: IX_media_record; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_media_record" ON public.media USING btree (record);


--
-- Name: IX_personal_bests_record; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_personal_bests_record" ON public.personal_bests USING btree (record);


--
-- Name: IX_personal_bests_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_personal_bests_user" ON public.personal_bests USING btree ("user");


--
-- Name: IX_player_points_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_player_points_user" ON public.player_points USING btree ("user");


--
-- Name: IX_records_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_records_user" ON public.records USING btree ("user");


--
-- Name: IX_upvotes_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_upvotes_user" ON public.upvotes USING btree ("user");


--
-- Name: IX_votes_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_votes_user" ON public.votes USING btree ("user");


--
-- Name: IX_world_records_record; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_world_records_record" ON public.world_records USING btree (record);


--
-- Name: IX_world_records_user; Type: INDEX; Schema: public; Owner: -
--

CREATE INDEX "IX_world_records_user" ON public.world_records USING btree ("user");


--
-- Name: auth auth_user_foreign; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.auth
    ADD CONSTRAINT auth_user_foreign FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: favorites favorites_user_foreign; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.favorites
    ADD CONSTRAINT favorites_user_foreign FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: levels levels_metadata_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.levels
    ADD CONSTRAINT levels_metadata_id_fkey FOREIGN KEY (metadata_id) REFERENCES public.metadata(id) NOT VALID;


--
-- Name: media media_record_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.media
    ADD CONSTRAINT media_record_fkey FOREIGN KEY (record) REFERENCES public.records(id);


--
-- Name: personal_bests personal_bests_record_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.personal_bests
    ADD CONSTRAINT personal_bests_record_fkey FOREIGN KEY (record) REFERENCES public.records(id);


--
-- Name: personal_bests personal_bests_user_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.personal_bests
    ADD CONSTRAINT personal_bests_user_fkey FOREIGN KEY ("user") REFERENCES public.users(id);


--
-- Name: player_points player_points_user_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.player_points
    ADD CONSTRAINT player_points_user_fkey FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: records records_user_foreign; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.records
    ADD CONSTRAINT records_user_foreign FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: upvotes upvotes_user_foreign; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.upvotes
    ADD CONSTRAINT upvotes_user_foreign FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: votes votes_user_foreign; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.votes
    ADD CONSTRAINT votes_user_foreign FOREIGN KEY ("user") REFERENCES public.users(id) ON DELETE SET NULL;


--
-- Name: world_records world_records_record_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.world_records
    ADD CONSTRAINT world_records_record_fkey FOREIGN KEY (record) REFERENCES public.records(id);


--
-- Name: world_records world_records_user_fkey; Type: FK CONSTRAINT; Schema: public; Owner: -
--

ALTER TABLE ONLY public.world_records
    ADD CONSTRAINT world_records_user_fkey FOREIGN KEY ("user") REFERENCES public.users(id);


--
-- PostgreSQL database dump complete
--
```
