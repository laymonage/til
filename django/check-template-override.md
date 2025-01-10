---
title: How to check if a Django template has been overridden
date: 2025-01-09
tags:
  - django
comments: true
description: >-
  If you're an author of a Django package, you might find it useful to check
  whether a template has been overridden. Here's a little snippet that helps you
  do that.
---

Django allows you to [override a template][overriding-templates] of one of your
installed apps.

If you're an author of a Django package, you might find it useful to check
whether one of your templates has been overridden. Here's a little snippet that
lets you do that.

```py
from pathlib import Path

from django.template import TemplateDoesNotExist
from django.template.loader import select_template


def template_is_overridden(
    template_name,
    expected_location,
    base_path=None,
):
    """Check if a Django template has been overridden."""
    try:
        template = select_template([template_name])
    except TemplateDoesNotExist:
        return False

    root = Path(base_path or __file__).resolve().parent
    path = str(root / expected_location / template_name)

    return template.origin.name != path
```

```py
>>> template_is_overridden(
        "wagtailadmin/generic/streamfield_block_preview.html",
        "wagtail/admin/templates",
    )
True
>>> template_is_overridden(
        "wagtail/admin/_do_not_touch.html",
        "wagtail/admin/templates",
    )
False
```

You can also specify the base path to be the root path of your app (or remove
the parameter and hardcode it instead).

```py
>>> from django.contrib import admin
>>> template_is_overridden(
        "admin/base.html",
        "templates",
        base_path=admin.__file__
    )
True
```

## Why would you need this?

<details>

<summary>Rationale</summary>

I've been working on [adding a preview feature][block-preview] to Wagtail's
StreamField block chooser. The feature will be enabled by default, but it needs
some configuration.

Requiring developers to configure a preview template for each block would be
cumbersome.

Instead, we provide a base skeleton template that developers can override and
extend to add the necessary static assets. It will then include the block's real
template fragment (which should already be configured in most cases).

We need some way to avoid enabling the feature when the developer hasn't
overridden the template. We can add a feature flag –a Django setting for
example– but it would be nice to have one less thing to configure.

We'll enable the feature only if the developer has either provided a specific
preview template for the block, or they have overridden the global template. The
former is easy to check, but the latter is where the above snippet helps.

I don't know if this is a hack and/or whether we'll actually use the check in
Wagtail. The feature is still in development, so
[I'm exploring some options][exploring-options]. Let me know your thoughts!

</details>

[overriding-templates]: https://docs.djangoproject.com/en/stable/howto/overriding-templates/
[block-preview]: https://github.com/wagtail/wagtail/pull/12700
[exploring-options]: https://github.com/wagtail/wagtail/pull/12700#discussion_r1909206156
