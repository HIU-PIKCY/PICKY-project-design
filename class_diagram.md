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
        - userId : Long
        - username : String
        - nickname : String
        - password : String
        - profileImage : String
        - loginType : LoginType
        - createdAt : LocalDateTime
        - updatedAt : LocalDateTime
        + validatePassword() Boolean
        + checkDuplicateUsername() Boolean
        + updateProfile() Void
        + changePassword() Void
    }
    class UserStats {
        - userStatsId : Long
        - userId : Long
        - totalBooks : Integer
        - totalQuestions : Integer
        - totalAnswers : Integer
        + updateStatistics() Void
        + getActivitySummary() Map~String, Object~
        + incrementBookCount() Void
        + incrementQuestionCount() Void
        + incrementAnswerCount() Void
    }
    class Book {
        - bookId : Long
        - title : String
        - author : String
        - publisher : String
        - isbn : String
        - coverImage : String
        - publishDate : LocalDateTime
        + searchByTitle() List~Book~
        + searchByAuthor() List~Book~
        + searchByTitleAndAuthor() List~Book~
        + getBookDetail() Book
    }
    class BookShelf {
        - bookShelfId : Long
        - userId : Long
        - domain : String
        + createShelf() BookShelf
        + updateShelf() Void
        + deleteShelf() Void
    }
    class MyBook {
        - myBookId : Long
        - userId : Long
        - bookId : Long
        - bookShelfId : Long
        - bookStatus : String
        + updateReadingStatus() Void
        + deleteMyBook() Void
        + getMyBooksByStatus() List~MyBook~
        + getRecentBooks() List~MyBook~
        + addToShelf() Void
        + removeFromShelf() Void
    }
    class AiQuestionTemplate {
        - templateId : Long
        - templateContent : String
        - isActive : Boolean
        - createdAt : LocalDateTime
        - updatedAt : LocalDateTime
        - systemId : Long
        - myBookId : Long
        - questionType : QuestionType
        + getTemplatesByCategory() List~AiQuestionTemplate~
        + generateQuestion() String
        + getActiveTemplates() List~AiQuestionTemplate~
    }
    class Question {
        - questionId : Long
        - myBookId : Long
        - questionContent : String
        - pageNum : Integer
        - questionTemplateId : Long
        - isAIGenerated : Boolean
        - createdAt : LocalDateTime
        - updatedAt : LocalDateTime
        - answerCount : Integer
        - likesCount : Integer
        + getQuestionsByBook() List~Question~
        + sortByLatest() List~Question~
        + sortByLikes() List~Question~
        + validateQuestion() Boolean
        + deleteQuestion() Void
    }
    class QuestionLikes {
        - questionLikeId : Long
        - userId : Long
        - questionId : Long
        - createdAt : LocalDateTime
        + likeQuestion() Void
        + unlikeQuestion() Void
        + isLikedByUser() Boolean
        + getLikeCount() Integer
    }
    class Answer {
        - answerId : Long
        - questionId : Long
        - userId : Long
        - answerTemplateId : Long
        - answerContent : String
        - isAIGenerated : Boolean
        - createdAt : LocalDateTime
        - updatedAt : LocalDateTime
        + createAnswer() Void
        + updateAnswer() Void
        + deleteAnswer() Void
        + getAnswerByQuestion() Answer
    }
    class AiAnswerTemplate {
        - aiTemplateId : Long
        - templateContent : String
        - isActive : Boolean
        - createdAt : LocalDateTime
        - updatedAt : LocalDateTime
        + getActiveTemplates() List~AiAnswerTemplate~
        + generateAnswerFromTemplate() String
        + getTemplateByCategory() AiAnswerTemplate
    }
    class Notification {
        - notificationId : Long
        - userId : Long
        - notificationType : String
        - content : String
        - isRead : Boolean
        - questionId : Long
        - answerId : Long
        + markAsRead() Void
        + getUnreadCount() Integer
        + createAnswerNotification() Void
        + getNotificationsByUser() List~Notification~
    }
    class System {
        - systemId : Long
        - systemKey : String
        - configValue : String
        - encrypted : Boolean
        - lastUpdated : LocalDateTime
        + getConfig() String
        + updateConfig() Void
    }
    %% ==========================================
    %% CONTROLLER LAYER (Presentation Layer)
    %% REST API 엔드포인트를 처리하는 컨트롤러 클래스
    %% ==========================================
    class AuthController {
        - userService : UserService
        + register() ResponseEntity~Object~
        + login() ResponseEntity~Object~
        + socialLogin() ResponseEntity~Object~
        + logout() ResponseEntity~Object~
        + validateUsername() ResponseEntity~Object~
    }
    class UserController {
        - userService : UserService
        - userStatsService : UserStatsService
        + getProfile() ResponseEntity~Object~
        + updateProfile() ResponseEntity~Object~
        + changePassword() ResponseEntity~Object~
        + uploadProfileImage() ResponseEntity~Object~
        + getActivitySummary() ResponseEntity~Object~
    }
    class BookController {
        - bookService : BookService
        - myBookService : MyBookService
        - bookShelfService : BookShelfService
        + searchBooks() ResponseEntity~Object~
        + searchByTitle() ResponseEntity~Object~
        + searchByAuthor() ResponseEntity~Object~
        + searchByTitleAndAuthor() ResponseEntity~Object~
        + getBookDetail() ResponseEntity~Object~
        + addToMyBooks() ResponseEntity~Object~
        + removeFromMyBooks() ResponseEntity~Object~
        + updateReadingStatus() ResponseEntity~Object~
        + getMyBooks() ResponseEntity~Object~
        + getMyBooksByStatus() ResponseEntity~Object~
        + manageShelves() ResponseEntity~Object~
    }
    class QuestionController {
        - questionService : QuestionService
        - aiQuestionTemplateService : AiQuestionTemplateService
        - questionLikesService : QuestionLikesService
        - aiService : AIService
        + getQuestions() ResponseEntity~Object~
        + getQuestionsByBook() ResponseEntity~Object~
        + createQuestion() ResponseEntity~Object~
        + deleteQuestion() ResponseEntity~Object~
        + generateAIQuestion() ResponseEntity~Object~
        + getQuestionTemplates() ResponseEntity~Object~
        + likeQuestion() ResponseEntity~Object~
        + unlikeQuestion() ResponseEntity~Object~
        + getQuestionLikes() ResponseEntity~Object~
        + sortQuestions() ResponseEntity~Object~
    }
    class AnswerController {
        - answerService : AnswerService
        - aiService : AIService
        + createAnswer() ResponseEntity~Object~
        + updateAnswer() ResponseEntity~Object~
        + deleteAnswer() ResponseEntity~Object~
        + generateAIAnswer() ResponseEntity~Object~
        + getAnswerByQuestion() ResponseEntity~Object~
    }
    class ActivityController {
        - activityService : ActivityService
        - userStatsService : UserStatsService
        + getMyActivities() ResponseEntity~Object~
        + getMyQuestions() ResponseEntity~Object~
        + getMyAnswers() ResponseEntity~Object~
        + getMyLikedQuestions() ResponseEntity~Object~
    }
    class NotificationController {
        - notificationService : NotificationService
        + getNotifications() ResponseEntity~Object~
        + markAsRead() ResponseEntity~Object~
        + getUnreadCount() ResponseEntity~Object~
        + markAllAsRead() ResponseEntity~Object~
    }
    %% ==========================================
    %% SERVICE LAYER (Business Logic Layer)
    %% 비즈니스 로직을 처리하는 서비스 클래스
    %% ==========================================
    class UserService {
        - userRepository : UserRepository
        + registerUser() User
        + loginUser() User
        + socialLogin() User
        + updateUserProfile() User
        + changePassword() Void
        + validateUsername() Boolean
        + uploadProfileImage() String
        + getUserById() User
    }
    class UserStatsService {
        - userStatsRepository : UserStatsRepository
        + updateUserStatistics() Void
        + getActivitySummary() UserStats
        + incrementBookCount() Void
        + incrementQuestionCount() Void
        + incrementAnswerCount() Void
        + createUserStats() UserStats
    }
    class BookService {
        - bookRepository : BookRepository
        + searchBooks() List~Book~
        + searchByTitle() List~Book~
        + searchByAuthor() List~Book~
        + searchByTitleAndAuthor() List~Book~
        + getBookById() Book
        + getBookDetail() Book
    }
    class BookShelfService {
        - bookShelfRepository : BookShelfRepository
        + createShelf() BookShelf
        + updateShelf() BookShelf
        + deleteShelf() Void
        + getShelvesByUser() List~BookShelf~
        + getShelfById() BookShelf
    }
    class MyBookService {
        - myBookRepository : MyBookRepository
        - userStatsService : UserStatsService
        - notificationService : NotificationService
        + addToMyBooks() MyBook
        + removeFromMyBooks() Void
        + updateReadingStatus() MyBook
        + getMyBooks() List~MyBook~
        + getMyBooksByStatus() List~MyBook~
        + getRecentBooks() List~MyBook~
        + addToShelf() Void
        + removeFromShelf() Void
    }
    class AiQuestionTemplateService {
        - aiQuestionTemplateRepository : AiQuestionTemplateRepository
        + getTemplatesByCategory() List~AiQuestionTemplate~
        + getActiveTemplates() List~AiQuestionTemplate~
        + createTemplate() AiQuestionTemplate
        + updateTemplate() AiQuestionTemplate
        + getTemplateById() AiQuestionTemplate
    }
    class QuestionService {
        - questionRepository : QuestionRepository
        - notificationService : NotificationService
        - userStatsService : UserStatsService
        + createQuestion() Question
        + deleteQuestion() Void
        + getQuestionsByBook() List~Question~
        + getQuestionsByUser() List~Question~
        + sortQuestions() List~Question~
        + sortByLatest() List~Question~
        + sortByLikes() List~Question~
        + validateQuestion() Boolean
        + getQuestionById() Question
    }
    class QuestionLikesService {
        - questionLikesRepository : QuestionLikesRepository
        + likeQuestion() QuestionLikes
        + unlikeQuestion() Void
        + isLikedByUser() Boolean
        + getLikeCount() Integer
        + getPopularQuestions() List~Question~
        + getLikedQuestionsByUser() List~Question~
    }
    class AnswerService {
        - answerRepository : AnswerRepository
        - notificationService : NotificationService
        - userStatsService : UserStatsService
        + createAnswer() Answer
        + updateAnswer() Answer
        + deleteAnswer() Void
        + getAnswerByQuestion() Answer
        + getAnswersByUser() List~Answer~
        + validateAnswer() Boolean
    }
    class AIService {
        - aiQuestionTemplateService : AiQuestionTemplateService
        - aiAnswerTemplateService : AiAnswerTemplateService
        + generateQuestion() String
        + generateAnswer() String
        + retryGeneration() String
        + generateFromTemplate() String
        + generateQuestionFromTemplate() String
        + generateAnswerFromTemplate() String
    }
    class AiAnswerTemplateService {
        - aiAnswerTemplateRepository : AiAnswerTemplateRepository
        + getActiveTemplates() List~AiAnswerTemplate~
        + getTemplateByCategory() AiAnswerTemplate
        + createTemplate() AiAnswerTemplate
        + updateTemplate() AiAnswerTemplate
        + getTemplateById() AiAnswerTemplate
    }
    class ActivityService {
        - questionRepository : QuestionRepository
        - answerRepository : AnswerRepository
        - questionLikesRepository : QuestionLikesRepository
        - userStatsRepository : UserStatsRepository
        + getMyQuestions() List~Question~
        + getMyAnswers() List~Answer~
        + getMyLikedQuestions() List~Question~
        + getActivitySummary() Map~String, Object~
        + getUserActivity() Map~String, Object~
    }
    class NotificationService {
        - notificationRepository : NotificationRepository
        + createNotification() Notification
        + createAnswerNotification() Notification
        + markAsRead() Void
        + markAllAsRead() Void
        + getUnreadCount() Integer
        + getNotificationsByUser() List~Notification~
        + deleteNotification() Void
    }
    %% ==========================================
    %% REPOSITORY LAYER (Data Access Layer)
    %% 데이터베이스 접근을 담당하는 레포지토리 클래스
    %% ==========================================
    class UserRepository {
        + findByUsername() Optional~User~
        + findById() Optional~User~
        + save() User
        + existsByUsername() Boolean
    }
    class UserStatsRepository {
        + findByUserId() Optional~UserStats~
        + save() UserStats
        + findById() Optional~UserStats~
    }
    class BookRepository {
        + findByTitleContaining() List~Book~
        + findByAuthor() List~Book~
        + findByTitleContainingOrAuthorContaining() List~Book~
        + findById() Optional~Book~
        + save() Book
    }
    class BookShelfRepository {
        + findByUserId() List~BookShelf~
        + findById() Optional~BookShelf~
        + save() BookShelf
        + delete() Void
    }
    class MyBookRepository {
        + findByUserIdAndBookId() Optional~MyBook~
        + findByUserIdOrderByCreatedAtDesc() List~MyBook~
        + findByUserIdAndBookStatus() List~MyBook~
        + findByUserId() List~MyBook~
        + save() MyBook
        + delete() Void
        + findById() Optional~MyBook~
    }
    class AiQuestionTemplateRepository {
        + findByQuestionType() List~AiQuestionTemplate~
        + findByIsActiveTrue() List~AiQuestionTemplate~
        + findAll() List~AiQuestionTemplate~
        + findById() Optional~AiQuestionTemplate~
        + save() AiQuestionTemplate
    }
    class QuestionRepository {
        + findByMyBookIdOrderByCreatedAtDesc() List~Question~
        + findByMyBookId() List~Question~
        + findByMyBookUserId() List~Question~
        + findByMyBookUserIdOrderByCreatedAtDesc() List~Question~
        + findById() Optional~Question~
        + save() Question
        + delete() Void
    }
    class QuestionLikesRepository {
        + findByUserIdAndQuestionId() Optional~QuestionLikes~
        + countByQuestionId() Integer
        + findByUserId() List~QuestionLikes~
        + findByUserIdOrderByCreatedAtDesc() List~QuestionLikes~
        + save() QuestionLikes
        + delete() Void
    }
    class AnswerRepository {
        + findByQuestionId() Optional~Answer~
        + findByQuestionMyBookUserId() List~Answer~
        + findByUserIdOrderByCreatedAtDesc() List~Answer~
        + findById() Optional~Answer~
        + save() Answer
        + delete() Void
    }
    class AiAnswerTemplateRepository {
        + findByIsActiveTrue() List~AiAnswerTemplate~
        + findByCategory() List~AiAnswerTemplate~
        + findById() Optional~AiAnswerTemplate~
        + save() AiAnswerTemplate
    }
    class NotificationRepository {
        + findByUserIdOrderByCreatedAtDesc() List~Notification~
        + countByUserIdAndIsReadFalse() Integer
        + findByUserId() List~Notification~
        + findById() Optional~Notification~
        + save() Notification
        + delete() Void
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
