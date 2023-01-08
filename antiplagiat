import argparse
from ast import parse, unparse, copy_location, Expr, FunctionDef, NodeTransformer, Name, Str


class NormFunction(NodeTransformer):
    def __init__(self):
        self.identifiers = {}
        super().__init__()

    def visit_FunctionDef(self, node):
        try:
            name = self.identifiers[node.name]
        except KeyError:
            name = f'function{len(self.identifiers):x}'
            self.identifiers[node.name] = name
        for i, arg in enumerate(node.args.args):
            arg.arg = f'arg{i}'
        new_func = FunctionDef(name=name, args=node.args, body=node.body, decorator_list=node.decorator_list)
        if isinstance(new_func.body[0], Expr) and isinstance(new_func.body[0].value, Str):
            del new_func.body[0]
        return copy_location(new_func, node)


class NormIdentifiers(NodeTransformer):
    def __init__(self):
        self.identifiers = {}
        super().__init__()

    def visit_Name(self, node):
        try:
            id_fr = self.identifiers[node.id]
        except KeyError:
            id_fr = f'id_{len(self.identifiers)}'
            self.identifiers[node.id] = id_fr
        return copy_location(Name(id=id_fr), node)


def get_norm(filename):
    with open(filename, encoding='utf-8') as f:
        tree = parse(f.read())
        tree = NormFunction().visit(tree)
        tree = NormIdentifiers().visit(tree)
        return unparse(tree)


def levenshtein(norm1, norm2):
    n = len(norm1)
    m = len(norm2)
    if n > m:
        n, m = m, n
        norm1, norm2 = norm2, norm1
    now = range(n + 1)
    for i in range(1, m + 1):
        prev, now = now, [i] + [0] * n
        for j in range(1, n + 1):
            now[j] = min(prev[j] + 1, now[j - 1] + 1, prev[j - 1] + (norm1[j - 1] != norm2[i - 1]))
    return now[n]


def get_stat(norm1, norm2):
    edit_dist = levenshtein(norm1, norm2)
    code_len = (len(norm1) + len(norm2)) / 2
    return 1 - edit_dist / code_len


def main():
    ap = argparse.ArgumentParser()
    ap.add_argument('files', nargs='+')
    args = ap.parse_args()
    with open(args.files[0], 'r') as f:
        lines = f.read().splitlines()
        stats = []
        for i in range(len(lines)):
            pair_py = lines[i].split()
            norm1 = get_norm(pair_py[0])
            norm2 = get_norm(pair_py[1])
            stats.append(get_stat(norm1, norm2))
    with open(args.files[1], 'w') as f:
        for stat in stats:
            f.write(f"{stat}\n")


if __name__ == '__main__':
    main()
