# SRS 8: Post Explorer

Post Explorer browses JSONPlaceholder posts, loads comments and authors, and lets users create posts locally. The profile icon opens the Firebase registration form from [Lab 7](README-LAB7.md).

## Run

```sh
flutter pub get
flutter pub run build_runner build
flutter run
```

## Architecture and API

The screens observe `PostsController` (ChangeNotifier). The controller calls `PostRepository`, which combines a SharedPreferences cache with a Retrofit client. A single Dio instance sets the base URL, timeouts, request headers, and logging interceptor. `AppException` distinguishes offline, timeout, client (4xx), server (5xx), JSON, and unknown errors.

Retrofit generates the typed API client. `json_serializable` converts Post, Comment, User, and nested Address models to and from JSON. Build Runner generates the `.g.dart` files.

| Request | Purpose |
| --- | --- |
| `GET /posts` with `_page`, `_limit` and optional `userId` | Fetch a page of posts, optionally filtered by author |
| `GET /posts/{id}` | Fetch a post |
| `GET /posts/{id}/comments` | Fetch its comments |
| `GET /users`, `GET /users/{id}` | Fetch authors or one author's details |
| `POST /posts` | Submit a post and display the JSON response |

The detail screen requests the post, its comments, and its author concurrently.

## Using the app

- The first page contains 10 posts. The list shows a loading indicator and offers retry on failure. On later visits it displays cached posts immediately, marks them as cached, and refreshes them.
- Each author has a separate cache key. Switching authors resets pagination and discards outdated responses. Network errors preserve the last successful cache. The app ignores malformed cache entries and can fetch fresh data.
- Search matches titles among loaded posts. The screen explains this limit and supports sorting by ID and resetting filters.
- Scroll to load the next page or pull to refresh. The controller tracks initial loading, refresh, and pagination separately. It blocks duplicate requests and lets users retry a failed page.
- To create a post, select an author and enter a title of 5 to 100 characters and a body of 10 to 1,000 characters. The form guards against duplicate submissions. JSONPlaceholder returns an example response with ID 101. Because the service does not persist writes, the app saves a separate local copy with a unique negative ID and labels it as locally created.

## Tests and assignment history

Run `flutter test test/posts_test.dart` for 11 offline tests covering JSON parsing and round trips, nested models, invalid JSON, repository and Retrofit requests, HTTP 500, author filtering, page deduplication, offline cache, POST through a mock Dio adapter, and stale responses. The project also includes three registration tests from Lab 7. [Full test output](docs/srs8-tests.txt)

The `srs-8` tag in the Post Explorer development history marks the stage before the SRS 9 changes.
