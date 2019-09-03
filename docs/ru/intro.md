# Для чего этот репозиторий?

Этот репозиторий содержит заготовки pdf для бизнес-планов и личных данных пользователя.

# Какова структура репозитория?

Структура репозитория должна быть следующая:
```
- LICENSE
- README.md
- docs/
  - ru/
    - intro.md
  - en/
    - intro.md
- business-plan-templates/
  - simple/
    - main.tex
  - <theme1>
    - main.tex
    - logo.jpg
    - aux.tex
  - <theme2>
    - main.tex
  ...
- profile-templates/
  - simple/
    - main.tex
  - <theme1>
    - main.tex
    - logo.jpg
    - aux.tex
  - <theme2>
    - main.tex
  ...
```
Всё, что относится к шаблону pdf, должно лежать в соответствующей директории.
Директория шаблона должна содержать main.tex - это основной файл шаблона. Также она может содержать и другие файлы, необходимые для правильной генерации pdf: картинки, другие шаблоны.

# Как выглядят шаблоны?

Все шаблоны - это некая помесь [latex](https://ru.wikipedia.org/wiki/LaTeX) и [jinja2](https://jinja.palletsprojects.com/en/2.10.x/). В том смысле, что они сначала будут прогоняться через jinja2 для подстановки значений, а затем уже отправляться на компиляцию в pdflatex. Пример шаблона вы можете найти в директории profile-templates/simple.

# Где найти названия полей?

Все названия полей будут соответствовать тем, которые есть в моделях и в админке(TBD) Django проекта [business_plan](https://gitlab.com/aido93/business_plan).

Например, в моделях Django есть такое:
```python
# plan/base_models.py
class BaseModel(models.Model):
    class Meta:
        abstract = True
    user             = models.ForeignKey('User', on_delete=models.CASCADE)

class BaseBusinessPlanComponent(models.Model):
    class Meta:
        abstract = True
    name             = models.CharField(_('name'), max_length=5000)
    created_by       = models.ForeignKey('User', null=True, blank=True, on_delete=models.SET_NULL)

class BaseBusinessPlanComponentGroup(BaseBusinessPlanComponent):
    class Meta:
        abstract = True
    business_plan    = models.ForeignKey('BusinessPlan',       on_delete=models.CASCADE)

# plan/models.py
class Pet(BaseModel):
    class Meta:
        verbose_name        = _('pet')
        verbose_name_plural = _('my pets')
    name             = models.CharField(_('name'), max_length=500)
    month_costs      = models.PositiveIntegerField(_('month costs'), default=0)
    def __str__(self):
        return self.name

class WorkersSet(BaseBusinessPlanComponentGroup):
    class Meta:
        verbose_name        = _('workers set')
        verbose_name_plural = _('workers sets')

# plan/admin.py
class WorkersSetAdmin(BaseBusinessPlanComponentAdmin):
    inlines = [ RolesInline, ]
    readonly_fields = ('salaries', 'search_on_avito')
    fields = ('name', 'business_plan', 'search_on_avito', 'salaries')
    list_display = ('name', 'business_plan')
    def salaries(self, obj):
        elements={}
        for worker in obj.role_set.all():
            mean_salary=(worker.min_salary+worker.max_salary)/2
            elements[worker.name]=mean_salary*worker.count
        return plot_pie(elements, 'workersSet', obj.id, _("Workers Set"))
    salaries.short_description=_('salaries')
admin.site.register(WorkersSet, WorkersSetAdmin)
```

Это означает, что в шаблоне отчета пользователя можно пользоваться значениями:
- `{{ user.pet[0].name }}`
- `{{ user.pet[0].month_costs }}`
- `{{ user.businessPlan[1].workersSet[0].name }}`
- `{{ user.businessPlan[1].workersSet[0].salaries }}`

То же самое касается и остальных моделей с более сложными ForeignKey-ами.
