app.py
    flask app
    SQLAlchemy
    imports blueprints from routes
    registers the blueprints with flast.register_blueprint(bp, url_prefix)

models.py
    SQLAlchemy(imported from app) association tables for: 
    friendships
    song_tags
    playlist_entries

    classes for: 
        User
        |id|username|email|listening_streak|last_listened_at|created_at|
        relationships:
            shared_songs one to many
            ratings one to many
            listening_events one to many
            notifications one to many
            playlists one to many
            friends many to many
        dict includes:
            id
            username
            listening_streak
            last_listened_at

        Tag
        |id|name|
        no relationships
        no dict

        Song
        |id|title|artist|album|genre|shared_by|shared_at|share_note|
        relationships:
            ratings one to many
            listening_events one to many
            tags many to many
        dict includes:
            id
            title
            artist
            album
            genre
            shared_by
            shared_at
            share_note
            tags

        ListeningEvent
        |id|user_id|song_id|listened_at|
        no relationships
        dict includes:
            id
            user_id
            song_id
            listened_at

        Rating
        |id|user_id|song_id|score|rated_at|
        user_id and song_id have unique_user_song_rating constraint
        no relationships
        dict includes:
            id
            user-id
            song_id
            score
            rated_at

        Playlist
        |id|name|created_by|created_at|is_collaborative|
        relationships:
            songs many to many
        dict includes:
            id
            name
            created_by
            created_at
            is_collaborative

        Notification
        |id|user_id|notification_type|body|created_at|read|
        dict includes
            id
            user_id
            type
            body
            created_at
            read

feed_service.py
    Handles the "Friends Listening Now" feed and activity feed logic.

    defines functions for:
        get_friends_listening_now(user_id)
        get_activity_feed(user_id, limit)


notification_service.py
    Handles creating and retrieving notifications.
    Notifications are generated when friends interact with a user's shared songs.
    
    defines functions for:
        create_notification(user_id, notification_type, body)
        add_to_playlist(playlist_id, song_id, added_by_user_id)
        rate_song(user_id, song_id, score)
        get_notifications(user_id, unread_only)
        mark_as_read(notification_id)

playlist_service.py
    Handles playlist creation and retrieval logic.

    defines functions for:
        create_playlist(name, created_by_user_id, is_collaborative)
        get_playlist_songs(playlist_id)
        get_playlist(playlist_id)
        get_user_playlists(user_id)

search_service.py
    Handles song search logic.

    defines functions for:
        search_songs(query)
        get_song(song_id)

streak_service.py
    Handles listening streak logic for users.
    A streak increments when a user listens on consecutive calendar days.
    It resets to 1 if a day is skipped.

    defines functions for:
        record_listening_event(user_id, song_id)
        update_listening_streak(user, now)
        get_streak(user_id)

routes/feed.py
    Blueprint: feed_bp, registered at url_prefix /feed
    Calls into feed_service

    defines routes for:
        GET /<user_id>/listening-now
        GET /<user_id>/activity

routes/playlists.py
    Blueprint: playlists_bp, registered at url_prefix /playlists
    Calls into playlist_service and notification_service

    defines routes for:
        POST /  (create playlist)
        GET /<playlist_id>
        GET /<playlist_id>/songs
        POST /<playlist_id>/songs  (add song to playlist)

routes/songs.py
    Blueprint: songs_bp, registered at url_prefix /songs
    Calls into search_service, notification_service, and streak_service

    defines routes for:
        GET /search
        GET /<song_id>
        POST /<song_id>/rate
        POST /<song_id>/listen

routes/users.py
    Blueprint: users_bp, registered at url_prefix /users
    Calls into streak_service and notification_service

    defines routes for:
        GET /<user_id>
        GET /<user_id>/streak
        GET /<user_id>/notifications
        POST /notifications/<notification_id>/read

seed_data.py
    Populates the database with realistic test data. Run with: python seed_data.py

    creates:
        5 users with established friendships
        25 songs with varying tag counts (0, 1, and 3+ tags)
        3 playlists with 5-10 songs each
        listening events spanning the past 2 weeks (including recent ones)
        existing streaks for some users
        existing playlist-add notifications

    defines functions for:
        seed()


bug reproduction:
    bug #1: by creating a session for a user that listened on saturday and then trying to update the streak on sunday, I found that the streak reset to 1.
    fix #1: removed "and today.weekday() != 6" from elif on line 73 of feed_service.py. Now properly increments even if today.weekday() returns sunday