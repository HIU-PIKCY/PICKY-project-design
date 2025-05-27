## 1. 사용자 회원가입 및 로그인

```mermaid
sequenceDiagram
    actor Client
    participant AuthController
    participant UserService
    participant UserRepository
    participant UserStatsService
    participant UserStatsRepository

    Client->>AuthController: register(userData)
    AuthController->>UserService: registerUser(userData)
    UserService->>UserRepository: existsByUsername(username)
    UserRepository-->>UserService: Boolean
    
    alt username exists
        UserService-->>AuthController: validation failed
    else username available
        UserService->>UserRepository: save(user)
        UserRepository-->>UserService: User
        UserService->>UserStatsService: createUserStats(userId)
        UserStatsService->>UserStatsRepository: save(userStats)
        UserStatsRepository-->>UserStatsService: UserStats
        UserStatsService-->>UserService: UserStats
        UserService-->>AuthController: User
    end
    
    AuthController-->>Client: ResponseEntity<Object>
```

## 2. 도서 검색 및 내 서재 추가

```mermaid
sequenceDiagram
    actor Client
    participant BookController
    participant BookService
    participant BookRepository
    participant MyBookService
    participant MyBookRepository
    participant UserStatsService
    participant UserStatsRepository

    Client->>BookController: searchByTitleAndAuthor(title, author)
    BookController->>BookService: searchByTitleAndAuthor(title, author)
    BookService->>BookRepository: findByTitleContainingOrAuthorContaining(title, author)
    BookRepository-->>BookService: List<Book>
    BookService-->>BookController: List<Book>
    BookController-->>Client: ResponseEntity<Object>

    Client->>BookController: addToMyBooks(userId, bookId)
    BookController->>MyBookService: addToMyBooks(userId, bookId)
    MyBookService->>MyBookRepository: findByUserIdAndBookId(userId, bookId)
    MyBookRepository-->>MyBookService: Optional<MyBook>
    
    alt MyBook doesn't exist
        MyBookService->>MyBookRepository: save(myBook)
        MyBookRepository-->>MyBookService: MyBook
        MyBookService->>UserStatsService: incrementBookCount(userId)
        UserStatsService->>UserStatsRepository: findByUserId(userId)
        UserStatsRepository-->>UserStatsService: Optional<UserStats>
        UserStatsService->>UserStatsRepository: save(userStats)
        UserStatsRepository-->>UserStatsService: UserStats
        UserStatsService-->>MyBookService: void
        MyBookService-->>BookController: MyBook
    else MyBook already exists
        MyBookService-->>BookController: duplicate book error
    end
    
    BookController-->>Client: ResponseEntity<Object>
```

## 3. 질문 생성 (AI 템플릿 사용)

```mermaid
sequenceDiagram
    actor Client
    participant QuestionController
    participant AIService
    participant AiQuestionTemplateService
    participant AiQuestionTemplateRepository
    participant QuestionService
    participant QuestionRepository
    participant UserStatsService
    participant UserStatsRepository

    Client->>QuestionController: generateAIQuestion(myBookId, questionType)
    QuestionController->>AIService: generateQuestion(myBookId, questionType)
    AIService->>AiQuestionTemplateService: getTemplatesByCategory(questionType)
    AiQuestionTemplateService->>AiQuestionTemplateRepository: findByQuestionType(questionType)
    AiQuestionTemplateRepository-->>AiQuestionTemplateService: List<AiQuestionTemplate>
    
    alt templates found
        AiQuestionTemplateService-->>AIService: List<AiQuestionTemplate>
        AIService->>AIService: generateQuestionFromTemplate(template)
        AIService-->>QuestionController: String
        
        QuestionController->>QuestionService: createQuestion(questionData)
        QuestionService->>QuestionRepository: save(question)
        QuestionRepository-->>QuestionService: Question
        QuestionService->>UserStatsService: incrementQuestionCount(userId)
        UserStatsService->>UserStatsRepository: findByUserId(userId)
        UserStatsRepository-->>UserStatsService: Optional<UserStats>
        UserStatsService->>UserStatsRepository: save(userStats)
        UserStatsRepository-->>UserStatsService: UserStats
        UserStatsService-->>QuestionService: void
        QuestionService-->>QuestionController: Question
    else no templates
        AiQuestionTemplateService-->>AIService: empty List
        AIService-->>QuestionController: generation failed
    end
    
    QuestionController-->>Client: ResponseEntity<Object>
```

## 4. 답변 작성 및 알림 생성

```mermaid
sequenceDiagram
    actor Client
    participant AnswerController
    participant AnswerService
    participant AnswerRepository
    participant NotificationService
    participant NotificationRepository
    participant UserStatsService
    participant UserStatsRepository

    Client->>AnswerController: createAnswer(answerData)
    AnswerController->>AnswerService: createAnswer(answerData)
    AnswerService->>AnswerRepository: save(answer)
    AnswerRepository-->>AnswerService: Answer
    AnswerService->>NotificationService: createAnswerNotification(questionId, answerId)
    NotificationService->>NotificationRepository: save(notification)
    NotificationRepository-->>NotificationService: Notification
    NotificationService-->>AnswerService: Notification
    AnswerService->>UserStatsService: incrementAnswerCount(userId)
    UserStatsService->>UserStatsRepository: findByUserId(userId)
    UserStatsRepository-->>UserStatsService: Optional<UserStats>
    UserStatsService->>UserStatsRepository: save(userStats)
    UserStatsRepository-->>UserStatsService: UserStats
    UserStatsService-->>AnswerService: void
    AnswerService-->>AnswerController: Answer
    AnswerController-->>Client: ResponseEntity<Object>
```

## 5. 질문 좋아요/취소

```mermaid
sequenceDiagram
    actor Client
    participant QuestionController
    participant QuestionLikesService
    participant QuestionLikesRepository

    Client->>QuestionController: likeQuestion(userId, questionId)
    QuestionController->>QuestionLikesService: isLikedByUser(userId, questionId)
    QuestionLikesService->>QuestionLikesRepository: findByUserIdAndQuestionId(userId, questionId)
    QuestionLikesRepository-->>QuestionLikesService: Optional<QuestionLikes>
    QuestionLikesService-->>QuestionController: Boolean
    
    alt not liked yet
        QuestionController->>QuestionLikesService: likeQuestion(userId, questionId)
        QuestionLikesService->>QuestionLikesRepository: save(questionLikes)
        QuestionLikesRepository-->>QuestionLikesService: QuestionLikes
        QuestionLikesService-->>QuestionController: QuestionLikes
    else already liked
        QuestionController->>QuestionLikesService: unlikeQuestion(userId, questionId)
        QuestionLikesService->>QuestionLikesRepository: delete(questionLikes)
        QuestionLikesRepository-->>QuestionLikesService: void
        QuestionLikesService-->>QuestionController: void
    end
    
    QuestionController-->>Client: ResponseEntity<Object>
```

## 6. 내 활동 조회

```mermaid
sequenceDiagram
    actor Client
    participant ActivityController
    participant ActivityService
    participant QuestionRepository
    participant AnswerRepository
    participant QuestionLikesRepository
    participant UserStatsRepository

    Client->>ActivityController: getMyActivities(userId)
    ActivityController->>ActivityService: getUserActivity(userId)
    
    par 병렬 조회
        ActivityService->>QuestionRepository: findByMyBookUserId(userId)
        QuestionRepository-->>ActivityService: List<Question>
    and
        ActivityService->>AnswerRepository: findByUserIdOrderByCreatedAtDesc(userId)
        AnswerRepository-->>ActivityService: List<Answer>
    and
        ActivityService->>QuestionLikesRepository: findByUserIdOrderByCreatedAtDesc(userId)
        QuestionLikesRepository-->>ActivityService: List<QuestionLikes>
    and
        ActivityService->>UserStatsRepository: findByUserId(userId)
        UserStatsRepository-->>ActivityService: Optional<UserStats>
    end
    
    ActivityService-->>ActivityController: Map<String, Object>
    ActivityController-->>Client: ResponseEntity<Object>
```

## 7. 알림 조회 및 읽음 처리

```mermaid
sequenceDiagram
    actor Client
    participant NotificationController
    participant NotificationService
    participant NotificationRepository

    Client->>NotificationController: getNotifications(userId)
    NotificationController->>NotificationService: getNotificationsByUser(userId)
    NotificationService->>NotificationRepository: findByUserIdOrderByCreatedAtDesc(userId)
    NotificationRepository-->>NotificationService: List<Notification>
    NotificationService-->>NotificationController: List<Notification>
    NotificationController-->>Client: ResponseEntity<Object>

    Client->>NotificationController: markAsRead(notificationId)
    NotificationController->>NotificationService: markAsRead(notificationId)
    NotificationService->>NotificationRepository: findById(notificationId)
    NotificationRepository-->>NotificationService: Optional<Notification>
    NotificationService->>NotificationRepository: save(notification)
    NotificationRepository-->>NotificationService: Notification
    NotificationService-->>NotificationController: void
    NotificationController-->>Client: ResponseEntity<Object>
```

## 8. AI 답변 생성

```mermaid
sequenceDiagram
    actor Client
    participant AnswerController
    participant AIService
    participant AiAnswerTemplateService
    participant AiAnswerTemplateRepository
    participant AnswerService
    participant AnswerRepository
    participant NotificationService
    participant NotificationRepository

    Client->>AnswerController: generateAIAnswer(questionId)
    AnswerController->>AIService: generateAnswer(questionId)
    AIService->>AiAnswerTemplateService: getActiveTemplates()
    AiAnswerTemplateService->>AiAnswerTemplateRepository: findByIsActiveTrue()
    AiAnswerTemplateRepository-->>AiAnswerTemplateService: List<AiAnswerTemplate>
    
    alt templates found
        AiAnswerTemplateService-->>AIService: List<AiAnswerTemplate>
        AIService->>AIService: generateAnswerFromTemplate(template, questionId)
        AIService-->>AnswerController: String
        
        AnswerController->>AnswerService: createAnswer(aiAnswerData)
        AnswerService->>AnswerRepository: save(answer)
        AnswerRepository-->>AnswerService: Answer
        AnswerService->>NotificationService: createAnswerNotification(questionId, answerId)
        NotificationService->>NotificationRepository: save(notification)
        NotificationRepository-->>NotificationService: Notification
        NotificationService-->>AnswerService: Notification
        AnswerService-->>AnswerController: Answer
    else no templates
        AiAnswerTemplateService-->>AIService: empty List
        AIService-->>AnswerController: generation failed
    end
    
    AnswerController-->>Client: ResponseEntity<Object>
```

## 9. 도서 상태 변경

```mermaid
sequenceDiagram
    actor Client
    participant BookController
    participant MyBookService
    participant MyBookRepository

    Client->>BookController: updateReadingStatus(myBookId, newStatus)
    BookController->>MyBookService: updateReadingStatus(myBookId, newStatus)
    MyBookService->>MyBookRepository: findById(myBookId)
    MyBookRepository-->>MyBookService: Optional<MyBook>
    
    alt myBook exists
        MyBookService->>MyBookRepository: save(myBook)
        MyBookRepository-->>MyBookService: MyBook
        MyBookService-->>BookController: MyBook
    else myBook not found
        MyBookService-->>BookController: update failed
    end
    
    BookController-->>Client: ResponseEntity<Object>
```

## 10. 서재 관리

```mermaid
sequenceDiagram
    actor Client
    participant BookController
    participant BookShelfService
    participant BookShelfRepository

    Client->>BookController: manageShelves(userId, shelfName, domain)
    BookController->>BookShelfService: createShelf(userId, shelfName, domain)
    BookShelfService->>BookShelfRepository: save(bookShelf)
    BookShelfRepository-->>BookShelfService: BookShelf
    BookShelfService-->>BookController: BookShelf
    BookController-->>Client: ResponseEntity<Object>

    Client->>BookController: getShelvesByUser(userId)
    BookController->>BookShelfService: getShelvesByUser(userId)
    BookShelfService->>BookShelfRepository: findByUserId(userId)
    BookShelfRepository-->>BookShelfService: List<BookShelf>
    BookShelfService-->>BookController: List<BookShelf>
    BookController-->>Client: ResponseEntity<Object>
```
