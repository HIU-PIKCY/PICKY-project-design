---
config:
  layout: elk
  theme: mc
  look: classic
---
classDiagram
    direction LR
    %% ==========================================
    %% MODEL LAYER (Entity Classes)
    %% ERD의 12개 테이블에 대응하는 엔티티 클래스
    %% ==========================================
    class User {
        -Long userId
        -String username
        -String nickname
        -String password
        -String profileImage
        -Enum loginType
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +validatePassword() boolean
        +checkDuplicateUsername() boolean
        +updateProfile() void
        +changePassword() void
    }
    class UserStats {
        -Long userStatsId
        -Long userId
        -Integer totalBooks
        -Integer totalQuestions
        -Integer totalAnswers
        +updateStatistics() void
        +getActivitySummary() Map
        +incrementBookCount() void
        +incrementQuestionCount() void
        +incrementAnswerCount() void
    }
    class Book {
        -Long bookId
        -String title
        -String author
        -String publisher
        -String isbn
        -String coverImage
        -LocalDateTime publishDate
        +searchByTitle() List
        +searchByAuthor() List
        +searchByTitleAndAuthor() List
        +getBookDetail() Book
    }
    class BookShelf {
        -Long bookShelfId
        -Long userId
        -String domain
        +createShelf() BookShelf
        +updateShelf() void
        +deleteShelf() void
    }
    class MyBook {
        -Long myBookId
        -Long userId
        -Long bookId
        -Long bookShelfId
        -String bookStatus
        +updateReadingStatus() void
        +deleteMyBook() void
        +getMyBooksByStatus() List
        +getRecentBooks() List
        +addToShelf() void
        +removeFromShelf() void
    }
    class AiQuestionTemplate {
        -Long templateId
        -String templateContent
        -Boolean isActive
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        -Long systemId
        -Long myBookId
        -Enum questionType
        +getTemplatesByCategory() List
        +generateQuestion() String
        +getActiveTemplates() List
    }
    class Question {
        -Long questionId
        -Long myBookId
        -String questionContent
        -Integer pageNum
        -Long questionTemplateId
        -Boolean isAIGenerated
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        -Integer answerCount
        -Integer likesCount
        +getQuestionsByBook() List
        +sortByLatest() List
        +sortByLikes() List
        +validateQuestion() boolean
        +deleteQuestion() void
    }
    class QuestionLikes {
        -Long questionLikeId
        -Long userId
        -Long questionId
        -LocalDateTime createdAt
        +likeQuestion() void
        +unlikeQuestion() void
        +isLikedByUser() boolean
        +getLikeCount() int
    }
    class Answer {
        -Long answerId
        -Long questionId
        -Long userId
        -Long answerTemplateId
        -String answerContent
        -Boolean isAIGenerated
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +createAnswer() void
        +updateAnswer() void
        +deleteAnswer() void
        +getAnswerByQuestion() Answer
    }
    class AiAnswerTemplate {
        -Long aiTemplateId
        -String templateContent
        -Boolean isActive
        -LocalDateTime createdAt
        -LocalDateTime updatedAt
        +getActiveTemplates() List
        +generateAnswerFromTemplate() String
        +getTemplateByCategory() AiAnswerTemplate
    }
    class Notification {
        -Long notificationId
        -Long userId
        -String notificationType
        -String content
        -Boolean isRead
        -Long questionId
        -Long answerId
        +markAsRead() void
        +getUnreadCount() int
        +createAnswerNotification() void
        +getNotificationsByUser() List
    }
    class System {
        -Long systemId
        -String systemKey
        -String configValue
        -Boolean encrypted
        -LocalDateTime lastUpdated
        +getConfig() String
        +updateConfig() void
    }
    %% ==========================================
    %% CONTROLLER LAYER (Presentation Layer)
    %% REST API 엔드포인트를 처리하는 컨트롤러 클래스
    %% ==========================================
    class AuthController {
        -UserService userService
        +register() ResponseEntity
        +login() ResponseEntity
        +socialLogin() ResponseEntity
        +logout() ResponseEntity
        +validateUsername() ResponseEntity
    }
    class UserController {
        -UserService userService
        -UserStatsService userStatsService
        +getProfile() ResponseEntity
        +updateProfile() ResponseEntity
        +changePassword() ResponseEntity
        +uploadProfileImage() ResponseEntity
        +getActivitySummary() ResponseEntity
    }
    class BookController {
        -BookService bookService
        -MyBookService myBookService
        -BookShelfService bookShelfService
        +searchBooks() ResponseEntity
        +searchByTitle() ResponseEntity
        +searchByAuthor() ResponseEntity
        +searchByTitleAndAuthor() ResponseEntity
        +getBookDetail() ResponseEntity
        +addToMyBooks() ResponseEntity
        +removeFromMyBooks() ResponseEntity
        +updateReadingStatus() ResponseEntity
        +getMyBooks() ResponseEntity
        +getMyBooksByStatus() ResponseEntity
        +manageShelves() ResponseEntity
    }
    class QuestionController {
        -QuestionService questionService
        -AiQuestionTemplateService aiQuestionTemplateService
        -QuestionLikesService questionLikesService
        -AIService aiService
        +getQuestions() ResponseEntity
        +getQuestionsByBook() ResponseEntity
        +createQuestion() ResponseEntity
        +deleteQuestion() ResponseEntity
        +generateAIQuestion() ResponseEntity
        +getQuestionTemplates() ResponseEntity
        +likeQuestion() ResponseEntity
        +unlikeQuestion() ResponseEntity
        +getQuestionLikes() ResponseEntity
        +sortQuestions() ResponseEntity
    }
    class AnswerController {
        -AnswerService answerService
        -AIService aiService
        +createAnswer() ResponseEntity
        +updateAnswer() ResponseEntity
        +deleteAnswer() ResponseEntity
        +generateAIAnswer() ResponseEntity
        +getAnswerByQuestion() ResponseEntity
    }
    class ActivityController {
        -ActivityService activityService
        -UserStatsService userStatsService
        +getMyActivities() ResponseEntity
        +getMyQuestions() ResponseEntity
        +getMyAnswers() ResponseEntity
        +getMyLikedQuestions() ResponseEntity
    }
    class NotificationController {
        -NotificationService notificationService
        +getNotifications() ResponseEntity
        +markAsRead() ResponseEntity
        +getUnreadCount() ResponseEntity
        +markAllAsRead() ResponseEntity
    }
    %% ==========================================
    %% SERVICE LAYER (Business Logic Layer)
    %% 비즈니스 로직을 처리하는 서비스 클래스
    %% ==========================================
    class UserService {
        -UserRepository userRepository
        +registerUser() User
        +loginUser() User
        +socialLogin() User
        +updateUserProfile() User
        +changePassword() void
        +validateUsername() boolean
        +uploadProfileImage() String
        +getUserById() User
    }
    class UserStatsService {
        -UserStatsRepository userStatsRepository
        +updateUserStatistics() void
        +getActivitySummary() UserStats
        +incrementBookCount() void
        +incrementQuestionCount() void
        +incrementAnswerCount() void
        +createUserStats() UserStats
    }
    class BookService {
        -BookRepository bookRepository
        +searchBooks() List
        +searchByTitle() List
        +searchByAuthor() List
        +searchByTitleAndAuthor() List
        +getBookById() Book
        +getBookDetail() Book
    }
    class BookShelfService {
        -BookShelfRepository bookShelfRepository
        +createShelf() BookShelf
        +updateShelf() BookShelf
        +deleteShelf() void
        +getShelvesByUser() List
        +getShelfById() BookShelf
    }
    class MyBookService {
        -MyBookRepository myBookRepository
        -UserStatsService userStatsService
        -NotificationService notificationService
        +addToMyBooks() MyBook
        +removeFromMyBooks() void
        +updateReadingStatus() MyBook
        +getMyBooks() List
        +getMyBooksByStatus() List
        +getRecentBooks() List
        +addToShelf() void
        +removeFromShelf() void
    }
    class AiQuestionTemplateService {
        -AiQuestionTemplateRepository aiQuestionTemplateRepository
        +getTemplatesByCategory() List
        +getActiveTemplates() List
        +createTemplate() AiQuestionTemplate
        +updateTemplate() AiQuestionTemplate
        +getTemplateById() AiQuestionTemplate
    }
    class QuestionService {
        -QuestionRepository questionRepository
        -NotificationService notificationService
        -UserStatsService userStatsService
        +createQuestion() Question
        +deleteQuestion() void
        +getQuestionsByBook() List
        +getQuestionsByUser() List
        +sortQuestions() List
        +sortByLatest() List
        +sortByLikes() List
        +validateQuestion() boolean
        +getQuestionById() Question
    }
    class QuestionLikesService {
        -QuestionLikesRepository questionLikesRepository
        +likeQuestion() QuestionLikes
        +unlikeQuestion() void
        +isLikedByUser() boolean
        +getLikeCount() int
        +getPopularQuestions() List
        +getLikedQuestionsByUser() List
    }
    class AnswerService {
        -AnswerRepository answerRepository
        -NotificationService notificationService
        -UserStatsService userStatsService
        +createAnswer() Answer
        +updateAnswer() Answer
        +deleteAnswer() void
        +getAnswerByQuestion() Answer
        +getAnswersByUser() List
        +validateAnswer() boolean
    }
    class AIService {
        -AiQuestionTemplateService aiQuestionTemplateService
        -AiAnswerTemplateService aiAnswerTemplateService
        +generateQuestion() String
        +generateAnswer() String
        +retryGeneration() String
        +generateFromTemplate() String
        +generateQuestionFromTemplate() String
        +generateAnswerFromTemplate() String
    }
    class AiAnswerTemplateService {
        -AiAnswerTemplateRepository aiAnswerTemplateRepository
        +getActiveTemplates() List
        +getTemplateByCategory() AiAnswerTemplate
        +createTemplate() AiAnswerTemplate
        +updateTemplate() AiAnswerTemplate
        +getTemplateById() AiAnswerTemplate
    }
    class ActivityService {
        -QuestionRepository questionRepository
        -AnswerRepository answerRepository
        -QuestionLikesRepository questionLikesRepository
        -UserStatsRepository userStatsRepository
        +getMyQuestions() List
        +getMyAnswers() List
        +getMyLikedQuestions() List
        +getActivitySummary() Map
        +getUserActivity() Map
    }
    class NotificationService {
        -NotificationRepository notificationRepository
        +createNotification() Notification
        +createAnswerNotification() Notification
        +markAsRead() void
        +markAllAsRead() void
        +getUnreadCount() int
        +getNotificationsByUser() List
        +deleteNotification() void
    }
    class UserRepository {
        +findByUsername() Optional
        +findById() Optional
        +save() User
        +existsByUsername() boolean
    }
    class UserStatsRepository {
        +findByUserId() Optional
        +save() UserStats
        +findById() Optional
    }
    class BookRepository {
        +findByTitleContaining() List
        +findByAuthor() List
        +findByTitleContainingOrAuthorContaining() List
        +findById() Optional
        +save() Book
    }
    class BookShelfRepository {
        +findByUserId() List
        +findById() Optional
        +save() BookShelf
        +delete() void
    }
    class MyBookRepository {
        +findByUserIdAndBookId() Optional
        +findByUserIdOrderByCreatedAtDesc() List
        +findByUserIdAndBookStatus() List
        +findByUserId() List
        +save() MyBook
        +delete() void
        +findById() Optional
    }
    class AiQuestionTemplateRepository {
        +findByQuestionType() List
        +findByIsActiveTrue() List
        +findAll() List
        +findById() Optional
        +save() AiQuestionTemplate
    }
    class QuestionRepository {
        +findByMyBookIdOrderByCreatedAtDesc() List
        +findByMyBookId() List
        +findByMyBookUserId() List
        +findByMyBookUserIdOrderByCreatedAtDesc() List
        +findById() Optional
        +save() Question
        +delete() void
    }
    class QuestionLikesRepository {
        +findByUserIdAndQuestionId() Optional
        +countByQuestionId() int
        +findByUserId() List
        +findByUserIdOrderByCreatedAtDesc() List
        +save() QuestionLikes
        +delete() void
    }
    class AnswerRepository {
        +findByQuestionId() Optional
        +findByQuestionMyBookUserId() List
        +findByUserIdOrderByCreatedAtDesc() List
        +findById() Optional
        +save() Answer
        +delete() void
    }
    class AiAnswerTemplateRepository {
        +findByIsActiveTrue() List
        +findByCategory() List
        +findById() Optional
        +save() AiAnswerTemplate
    }
    class NotificationRepository {
        +findByUserIdOrderByCreatedAtDesc() List
        +countByUserIdAndIsReadFalse() int
        +findByUserId() List
        +findById() Optional
        +save() Notification
        +delete() void
    }

    %% ==========================================
    %% 엔티티 관계 (ERD 기반)
    %% ==========================================
    
    User --> UserStats
    User --> BookShelf
    User --> MyBook
    User --> QuestionLikes
    User --> Answer
    User --> Notification
    
    Book --> MyBook
    BookShelf --> MyBook
    MyBook --> Question
    MyBook --> AiQuestionTemplate
    
    Question --> Answer
    Question --> QuestionLikes
    Question --> Notification
    Question --> AiQuestionTemplate
    
    Answer --> Notification
    Answer --> AiAnswerTemplate
    
    AiQuestionTemplate --> System

    %% ==========================================
    %% 계층 간 의존성 관계
    %% ==========================================
    
    %% Controller to Service 의존성
    AuthController --> UserService
    UserController --> UserService
    UserController --> UserStatsService
    BookController --> BookService
    BookController --> MyBookService
    BookController --> BookShelfService
    QuestionController --> QuestionService
    QuestionController --> AiQuestionTemplateService
    QuestionController --> QuestionLikesService
    QuestionController --> AIService
    AnswerController --> AnswerService
    AnswerController --> AIService
    ActivityController --> ActivityService
    ActivityController --> UserStatsService
    NotificationController --> NotificationService

    %% Service to Service 의존성
    MyBookService --> UserStatsService
    MyBookService --> NotificationService
    QuestionService --> NotificationService
    QuestionService --> UserStatsService
    AnswerService --> NotificationService
    AnswerService --> UserStatsService
    AIService --> AiQuestionTemplateService
    AIService --> AiAnswerTemplateService

    %% Service to Repository 의존성
    UserService --> UserRepository
    UserStatsService --> UserStatsRepository
    BookService --> BookRepository
    BookShelfService --> BookShelfRepository
    MyBookService --> MyBookRepository
    AiQuestionTemplateService --> AiQuestionTemplateRepository
    QuestionService --> QuestionRepository
    QuestionLikesService --> QuestionLikesRepository
    AnswerService --> AnswerRepository
    AiAnswerTemplateService --> AiAnswerTemplateRepository
    NotificationService --> NotificationRepository
    ActivityService --> QuestionRepository
    ActivityService --> AnswerRepository
    ActivityService --> QuestionLikesRepository
    ActivityService --> UserStatsRepository