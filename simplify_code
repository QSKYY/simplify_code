import ast
import autopep8

class CodeSimplifier(ast.NodeTransformer):
    def __init__(self):
        self.used_names = set()
        self.imports_to_keep = set()

    def visit_FunctionDef(self, node):
        # Удаление функций, которые не используются
        if node.name not in self.used_names:
            return None
        # Удаление пустых функций
        if not node.body:
            return None
        return self.generic_visit(node)

    def visit_Import(self, node):
        # Добавляем импортируемые модули в список, если они используются
        for alias in node.names:
            if alias.name in self.used_names:
                self.imports_to_keep.add(alias.name)
        # Удаляем неиспользуемые импорты
        if not self.imports_to_keep.intersection(alias.name for alias in node.names):
            return None
        return node

    def visit_ImportFrom(self, node):
        # Добавляем импортируемые модули из "from" в список, если они используются
        for alias in node.names:
            if alias.name in self.used_names:
                self.imports_to_keep.add(alias.name)
        if not self.imports_to_keep.intersection(alias.name for alias in node.names):
            return None
        return node

    def visit_Assign(self, node):
        # Удаление неиспользуемых переменных
        targets = [t.id for t in node.targets if isinstance(t, ast.Name)]
        if not any(name in self.used_names for name in targets):
            return None
        return node

    def visit_Name(self, node):
        # Добавляем все используемые имена (переменные и функции) в список
        if isinstance(node.ctx, ast.Load):
            self.used_names.add(node.id)
        return node

    def visit_If(self, node):
        # Упрощаем условие, если оно заведомо истинное или ложное
        if isinstance(node.test, ast.Constant):
            if node.test.value:  # Если условие всегда истинно
                return node.body
            else:  # Если условие всегда ложно
                return None
        return self.generic_visit(node)

    def simplify_code(self, code):
        # Парсинг кода и создание дерева синтаксического анализа
        tree = ast.parse(code)
        
        # Первый проход: собираем все используемые имена
        self.visit(tree)
        
        # Повторный проход для удаления неиспользуемых элементов
        tree = self.generic_visit(tree)
        
        # Преобразование дерева обратно в код
        simplified_code = ast.unparse(tree)
        
        # Форматирование и минимизация кода
        formatted_code = autopep8.fix_code(simplified_code)
        
        return formatted_code

# Пример использования
code = """
import math
import os

def unused_function():
    pass

def calculate_area(radius):
    return math.pi * radius ** 2

result = calculate_area(5)
print(result)
"""

simplifier = CodeSimplifier()
simplified_code = simplifier.simplify_code(code)
print(simplified_code)
