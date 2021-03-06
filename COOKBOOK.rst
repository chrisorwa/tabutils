tabutils Cookbook
=================

Note that the ``pandas`` equivalent methods are preceded by ``-->``.
Command output is preceded by ``>>>``.

.. code-block:: python

    import itertools as it
    import pandas as pd

    from random import random
    from tabutils import io, process as pr, convert as cv
    from io import StringIO

    # To setup, lets define a universal header
    header = ['A', 'B', 'C', 'D']

    # Create some data in the same structure as what the various `read...`
    # functions output
    data = [(random.random() for _ in range(4)) for x in range(7)]
    records = [dict(zip(header, d)) for d in data]
    records[0]
    >>> {'A': 0.63872..., 'B': 0.89517..., 'C': 0.11175..., 'D': 0.49445...}

    """Select multiple columns"""
    next(pr.cut(records, ['A', 'B'], exclude=True))
    >>> {'C': 0.11175001869696033, 'D': 0.4944504196475903}

    # Now create some pieces to concatenate
    pieces = [it.islice(records, 3), it.islice(records, 3, None)]

    """Concatenate records together --> pd.concat(pieces)"""
    concated = it.chain(*pieces)
    next(concated)
    >>> {'A': 0.63872..., 'B': 0.89517..., 'C': 0.11175..., 'D': 0.49445...}
    len(list(concated)) + 1
    >>> 7

    # Now let's create two sets of records that we want to join
    left = [{'key': 'foo', 'lval': 1}, {'key': 'foo', 'lval': 2}]
    right = [{'key': 'foo', 'rval': 4}, {'key': 'foo', 'rval': 5}]

    """SQL style joins --> pd.merge(left, right, on='key')"""
    list(pr.join(left, right))
    >>> [
    ... {'key': 'foo', 'lval': 1, 'rval': 4},
    ... {'key': 'foo', 'lval': 1, 'rval': 5},
    ... {'key': 'foo', 'lval': 2, 'rval': 4},
    ... {'key': 'foo', 'lval': 2, 'rval': 5}]

    # Now let's create a new records-like list
    records = [
        {'A': 'foo', 'B': -1.202872},
        {'A': 'bar', 'B': 1.814470},
        {'A': 'foo', 'B': 1.8028870},
        {'A': 'bar', 'B': -0.595447}]

    """Group and sum"""
    # Select a function to be applied to the records contained in each group
    # In this case, we want to merge the records by summing field `B`.
    kwargs = {'aggregator': pr.merge, 'pred': 'B', 'op': sum}

    # Now group `records` by the value of field `A`, and pass `kwargs` which contains
    # details on the function to apply to each group of records.
    list(pr.group(records, 'A', **kwargs)
    >>> [{'A': 'bar', 'B': 1.219023}, {'A': 'foo', 'B': 0.600015}]

    # Now lets generate some random data to manipulate
    rrange = random.sample(range(-10, 10), 12)
    a = ['one', 'one', 'two', 'three'] * 3
    b = ['ah', 'beh', 'say'] * 4
    c = ['foo', 'foo', 'foo', 'bar', 'bar', 'bar'] * 2
    d = (random.random() * x for x in rrange)
    values = zip(a, b, c, d)
    records = (dict(zip(header, v)) for v in values)

    # Since `records` is an iterator over the rows, we have to be careful not
    # to inadvertently consume it. Lets use `pr.peek` to view the first few rows
    records, peek = pr.peek(records)
    peek
    >>> [
    ... {'A': 'one', 'B': 'ah', 'C': 'foo', 'D': -3.69822},
    ... {'A': 'one', 'B': 'beh', 'C': 'foo', 'D': -3.72018},
    ... {'A': 'two', 'B': 'say', 'C': 'foo', 'D': 1.02146},
    ... {'A': 'three', 'B': 'ah', 'C': 'bar', 'D': 0.38015},
    ... {'A': 'one', 'B': 'beh', 'C': 'bar', 'D': 0.0}]

    """Pivot tables
    --> pd.pivot_table(records, values='D', index=['A', 'B'], columns=['C'])
    """
    # Let's create a classic excel style pivot table
    pivot = pr.pivot(records, 'D', 'C')
    pivot, peek = pr.peek(pivot)
    peek
    >>> [
    ... {'A': 'one', 'B': 'ah', 'bar': 2.23933, 'foo': -3.69822},
    ... {'A': 'one', 'B': 'beh', 'bar': 0.0, 'foo': -3.72018},
    ... {'A': 'one', 'B': 'say', 'bar': 2.67595, 'foo': -5.55774},
    ... {'A': 'three', 'B': 'ah', 'bar': 0.38015},
    ... {'A': 'three', 'B': 'beh', 'foo': 5.79430}]

    """Data normalization --> pivot.stack()"""
    # To get the data back to its original form, we must normalize it.
    normal = pr.normalize(pivot, 'D', 'C', ['foo', 'bar'])
    normal, peek = pr.peek(normal)
    peek
    >>> [
    ... {'A': 'one', 'B': 'ah', 'C': 'foo', 'D': -3.69822},
    ... {'A': 'one', 'B': 'ah', 'C': 'bar', 'D': 2.23933},
    ... {'A': 'one', 'B': 'beh', 'C': 'foo', 'D': -3.72018},
    ... {'A': 'one', 'B': 'beh', 'C': 'bar', 'D': 0.0},
    ... {'A': 'one', 'B': 'say', 'C': 'foo', 'D': -5.55774}]
