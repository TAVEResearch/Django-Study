- django-taggit, django-taggit-templatetags2 사용

1. 애플리케이션 설계
    - 각 포스트마다 태그
    - 특정 태그를 가진 모든 포스트의 리스트
    - 태그 클라우드 기능(태그만 모아서 보여줌)
    
    1. 화면 UI 설계
    2. 테이블 설계
        - tags — TaggableManager(타입) — Blank, Null(제약조건)
    3. URL 설계
        - /blog/tag/ :: TagCloudTV(TemplateView), taggit_cloud.html
        - /blog/tag/tagname/ :: TaggedObjectLV(ListView), taggit_post_list.html
        - TemplateView, ListView.
    4. 작업/코딩 순서
        
        → settings.py에 taggit, taggit_templatetags2 앱 등록
        
        → models.py에 tags 필드 추가
        
        → admin.py에 태그 관련 내용 추가
        
        → makemigrations, migrate
        
        → urls.py, views.py에 로직 추가
        
        → templates 디렉터리에 템플릿 파일 수정 및 추가
        
    
2. 개발 코딩
    
    taggit 등록, 파일 수정
    
    1. 설정 파일에 taggit 앱 등록
        - 설정파일에 등록할지, 그리고 등록한다면 어떤 이름으로 등록할지를 확인해야.
        - 패키지 공식 문서에서 확인 가능
            - https://django-taggit.readthedocs.io/en/latest/index.html
            - https://github.com/fizista/django-tggit-templatetags2
        - settings.py
        
    2. 모델 코딩
        - tags 추가 to models.py
        - 타입 - TaggableManager :: **ManyToManyField, models.Manager** 역할을 동시에 함
            
            Tags라는 별칭, null = True 옵션 디폴트 설정
            
            - 우리의 코드
            
            ```python
            class Post(models.Model):
                title = models.CharField(verbose_name='TITLE', max_length=50)
                slug = models.SlugField('SLUG', unique=True, allow_unicode=True, help_text='one word for title alias.')
                description = models.CharField('DESCRIPTION', max_length=100, blank=True, help_text='simple description text.')
                content = models.TextField('CONTENT')
                create_dt = models.DateTimeField('CREATE DATE', auto_now_add=True)
                modify_dt = models.DateTimeField('MODIFY DATE', auto_now=True)
                tags = TaggableManager(blank=True)
            ```
            
            - models.Manager가 뭔지
                - [https://docs.djangoproject.com/en/4.0/topics/db/managers/](https://docs.djangoproject.com/en/4.0/topics/db/managers/)
                
                ```python
                # First, define the Manager subclass.
                class DahlBookManager(models.Manager):
                    def get_queryset(self):
                        return super().get_queryset().filter(author='Roald Dahl')
                
                # Then hook it into the Book model explicitly.
                class Book(models.Model):
                    title = models.CharField(max_length=100)
                    author = models.CharField(max_length=50)
                
                    objects = models.Manager() # The default manager.
                    dahl_objects = DahlBookManager() # The Dahl-specific manager.
                ```
                
                - Book.objects.all() → 모든 book, **Book.dahl_objects.all() → Roald Dahl의 book들.**
                - Book이라는 모델(테이블)의 object들을 불러오는 데에 사용하는 메소드인 get_queryset을 새롭게 짤 수 있음.
                - 조건에 맞추어 새롭게 object를 긁을 수 있게 된다.
                
            - ManyToManyField는 뭐죠
                - Many to many : 다대다 관계. 하나의 엔티티와 다른 하나의 엔티티의 Foreign Key만으로 DB 정의가 안 되고, 새로운 조인 테이블을 정의해야 함.
                - 즉 이 속성이 있다는 것 자체가 하나의 테이블을 새로 만드는 거라고 생각할 수 있다
                
            - tag같은 경우, Post 하나에 여러 개의 tag가 할당될 수 있고 tag 하나에도 여러 개의 Post가 할당될 수 있으므로 Many to many에 해당됨
            - 그래서 tag를 위한 새로운 조인 테이블을 만들어야 함(ManyToManyField)
                
                 + tag와 관련된 object들을 긁어오기 위한 Manager도 새로 정의(models.Manager) 했다고 볼 수 있을 듯!
                
            
        - admins.py에 tags 관련 내용 추가
            - 우리 코드
            
            ```python
            class PostAdmin(admin.ModelAdmin):
                list_display = ('id', 'title', 'modify_dt', 'tag_list')
                list_filter = ('modify_dt',)
                search_fields = ('title', 'content')
                prepopulated_fields = {'slug':('title',)}
            
                def get_queryset(self, request):
                    return super().get_queryset(request).prefetch_related('tags')
            
                def tag_list(self, obj):
                    return ', '.join(o.name for o in obj.tags.all())
            ```
            
            - 여기서의 코드 동작은 정확히는 모르겠지만.
            - get_queryset
                - super().get_queryset(request) : 원래 긁어오는 object들, 즉 Post들의 리스트가 될 것
                - .prefetch_related(’tags’) :
                    
                    [https://wave1994.tistory.com/70](https://wave1994.tistory.com/70)
                    
                    ForeignKey를 고려해서 조인 테이블을 만들어 줌.
                    
                    블로그 내의 코드 - prefetch_related 쪽 확인
                    
                    ```python
                    # 1
                    Person.objects.get(id=4).language.values()
                    # 2
                    Person.objects.prefetch_related('language').get(id=4).language.values()
                    ```
                    
                    1번
                    
                    - Person.objects.get(id=4) ?? Person 테이블의 4번 객체를 가져오겠다.
                    - id=4인 객체의 language 어트리뷰트의, values()를 가져와라.
                    
                    ```python
                    language = models.ManyToManyField('Language', through='PersonLanguage')
                    ```
                    
                    - language는 ‘PersonLanguage’라는 조인 테이블을 통해, Language와 ManyToMany로 연결되어 있다.
                    - → language 리스트를 가져옴.
                    - 이 과정이, get(id=4), 와 language.values()의 두 개 작업이므로 두 번 DB에 접근해야 한다는 게 비효율 초래. 그래서 prefetch_related를 사용한다.
                    
                    2번
                    
                    - Person.objects.prefetch_related(’language’) 테이블의 4번 객체의,
                    - , language 어트리뷰트의, values()를 가져와라
                    - 첫 번째 단계에서 처음부터 Person과 Language와의 조인 테이블을 만들어서 긁어오기 때문에 한 번의 작업만으로 똑같은 일을 한다.
                
            - 우리 코드는 get_queryset()함수가 super().get_queryset(request).prefetch_related(’tags’) 로 오버라이딩 되었음.
            - tag가 ManyToManyField로 Post와 연결되어 있으니까, Post와 tags의 조인 테이블을 만들어서 가져오도록 하는 게 새로운 get_queryset() 함수인 것!
            - 그런데 코드를 지우면 Admin페이지에 뭔가 변화가 나타날 줄 알았는데 모르겠음. 어디에 쓰이는 건지 모르겠네요...
            
            - def tag_list()는, Post 하나의 object에 대해 그거랑 연결된 tags를 전부 모아서, 리스트로 출력함. : 이게 Admin페이지의 각 Post에 연결된 tags를 모아서 네 번째 컬럼에 보여주는 것
            
    3. URLconf 코딩
        - 코드
        
        ```python
        # Example : /blog/tag/
        path('tag/', views.TagCloudTV.as_view(), name='tag_cloud'),
        
        # Example : /blog/tag/tagname/
        path('tag/<str:tag>/', views.TaggedObjectLV.as_view(), name='tagged_object_list'),
        ```
        
        - 두 개의 path가 추가되었음.
        - tag/ :: TagCloudTV. → 저번 주에 있었던 TemplateView를 사용, 그냥 기본 template를 보여주기 위함. views에서 taggit/taggit_cloud.html을 보여주게 설정되어 있음
        - tag/<str:tag>/ :: tag 이름이 주어질 경우, TaggedObjectLV를 이용해 template 보여줌.
        
    4. View 코딩
        - 추가된 두 코드
        
        ```python
        class TagCloudTV(TemplateView):
            template_name = 'taggit/taggit_cloud.html'
        
        class TaggedObjectLV(ListView):
            template_name = 'taggit/taggit_post_list.html'
            model = Post
        
            def get_queryset(self):
                return Post.objects.filter(tags__name=self.kwargs.get('tag'))
        
            def get_context_data(self, **kwargs):
                context = super().get_context_data(**kwargs)
                context['tagname'] = self.kwargs['tag']
                return context
        ```
        
        - TagCloudTV는 그냥 Template 보여주는 게 다임, {% get_tagcloud as tags %} → tags라는 이름으로 tagcloud를 가져오겠다. 아마 taggit 라이브러리에 내장된 기능일 듯
        - TaggedObjectLV :
            - get_queryset :
                - Post object들 중, tag name이, URLconf에서 가져온 (path('tag/<str:tag>/', ...) tag와 일치하는 것들만. Post에서 가져와라.
                - tags__name이 정확히 뭘 의미하는지는 잘 모르겠지만.. 아마 tags라는 속성의 이름을 표시할 때 사용하는 듯. self.kwargs.get(’tag’)는 URLconf에서 가져온 파라미터들 중 ‘tag’로 들어간 값을 가져오는 거로 추측
            - get_context_data :
                - 기존에 ListView에서 받았던 context는 object_list, 즉 객체의 리스트이다.
                - context = {’object_list’ : <객체 리스트>} 였다는 거. 이게 super().get_context_data(**kwargs)
                - context[’tagname’]에 tag이름을 넣는다.
                - 이러면 context = {’object_list’ : <객체 리스트>, ‘tagname’ : <태그 이름>}이 될 것. object_list와 tagname은 taggit/taggit_post_list.html에서 컨텍스트 변수로 활용될 것
                
            - self.kwargs.get(’tag’)와 self.kwargs[’tag’]의 차이는 뭘까? 비슷한 기능인 듯 한데 get_queryset에서는 get(’tag’)를 사용, get_context_data에서는 [’tag’]를 사용.
                - 수정해서 확인해 봤는데 같은 기능인 듯, 에러가 안 떴음
                
    5. 템플릿 코딩
        - 수정/생성 템플릿 목록
            - taggit/taggit_cloud.html
            - taggit/taggit_post_list.html
            - post_detail.html
        
        - post_detail.html
            - 포스트 글 하단에 태그를 표시
            
            ```python
            <b>TAGS</b> <i class="fas fa-tag"></i>
                {% load taggit_templatetags2_tags %}
                {% get_tags_for_object object as "tags" %}
                {% for tag in tags %}
                <a href="{% url 'blog:tagged_object_list' tag.name %}">{{ tag.name }}</a>
                {% endfor %}
                &emsp;
                <a href="{% url 'blog:tag_cloud' %}"><span class="btn btn-info btn-sm">TagCloud</span></a>
            ```
            
            - {% load taggit_templatetags2_tags %} : 여기 패키지에 정의된 태그 사용을 위해... 로딩
            - {% get_tags_for_object object as “tags” %} : Post의 하나의 object에 달려 있는 태그 리스트를 tags 템플릿 변수에 할당. (”” 없어도 잘 되나? → 잘 됨. 왜이렇게 문법이 일관성이 없는지)
            - {% for tag in tags %} : tags의 각 요소를 순회하면서, tag.name을 출력
            - 링크는, blog:tagged_object_list url에, tag.name을 인자로 넘기면서 지정
            - 마지막에 tag_cloud 템플릿으로 가는 링크를 추가, 버튼은 부트스트랩으로 만듦.
        
        - taggit_cloud.html
            - 코드
            
            ```python
            <div class="tag-cloud">
                    {% load taggit_templatetags2_tags %}
                    {% get_tagcloud as tags %}
                    {% for tag in tags %}
                    <span class="tag-{{tag.weight|floatformat:0}}">
                    <a href="{% url 'blog:tagged_object_list' tag.name %}"> {{tag.name}}({{tag.num_times}})</a>
                    </span>
                    {% endfor %}
                </div>
            ```
            
            - taggit_templatetags2_tags 임포트
            - {% get_tagcloud as tags %} : 커스텀 태그, 이걸로 모든 태그를 추출해서 tags 템플릿 변수에 할당
            - {% for tag in tags %} : tags를 순회하면서 링크를 만든다. tagged_object_list로 가는 tag, tag.name에다가.
            - tag.name : 태그 이름
            - tag.num_times : 태그가 몇 번 사용되었는지
            - tag.weight : 태그의 중요도. 자세한 내용은 알아서 정의되어 있나 봄.
            - tag.weight | floatformat:0 : tag.weight 값이 실수형이므로, 반올림해서 정수형으로 반환. 0이면 0번째 자리까지 보여준다는 뜻인가?
            
        - taggit_post_list.html
            - object_list와 tagname이 context 변수로 들어감.
            - 이거만 알면 이해 안될게 딱히 없어서 넘어갑니다.
