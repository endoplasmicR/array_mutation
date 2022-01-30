import json

ADD = "$add"
REMOVE = "$remove"
UPDATE = "$update"


class Mention:
    next_mention_id: int = 0

    def __init__(self,
                 obj:dict):
        self.id = obj.get("_id", None)
        self.text = obj.get("text", None)
        if self.id:
            Mention.next_mention_id = max(Mention.next_mention_id, self.id)+1

    def get_rep(self) -> str:
        return json.dumps(self, default=lambda o: o.__dict__, indent=2)


class Post:
    next_post_id: int = 0

    def __init__(self,
                 obj: dict):
        self.id = obj.get("_id", None)
        self.value = obj.get("value", None)
        if self.id:
            Post.next_post_id = max(Post.next_post_id, self.id)+1

        self.mentions = []
        self.id_to_mention = {}
        for m in obj.get("mentions", []):
            men = Mention(m)
            self.mentions.append(men)
            self.id_to_mention[men.id] = men

    def add_mention(self,
                    mention: Mention):
        mention.id = Mention.next_mention_id
        Mention.next_mention_id += 1
        self.id_to_mention[mention.id] = mention
        self.mentions.append(mention)

    def get_rep(self) -> str:
        return json.dumps(self, default=lambda o: o.__dict__, indent=2)


class ContentCreator:
    def __init__(self,
                 obj: dict):
        self.id = obj.get("_id", None)
        self.name = obj.get("name", None)
        self.posts = []
        self.id_to_post = {}
        for p in obj.get("posts", []):
            existing_post = Post(p)
            self.posts.append(existing_post)
            self.id_to_post[existing_post.id] = existing_post

    def add_post(self,
                 post: Post):
        post.id = Post.next_post_id
        Post.next_post_id += 1

        self.posts.append(post)
        self.id_to_post[post.id] = post

    def get_rep(self) -> str:
        return json.dumps(self, default=lambda o: o.__dict__, indent=2)


original_content_creator = ContentCreator({})


def generate_update_statement(
        original_doc: str,
        mutation_doc: str,
        ) -> str:
    global original_content_creator
    original_content_creator = ContentCreator(json.loads(original_doc))
    mutation = json.loads(mutation_doc)

    mutated_posts = mutation.get("posts", [])
    update_result = {}

    updated = {}
    added = {}
    removed = {}

    for mutated_post in mutated_posts:
        original_id = mutated_post.get("_id", None)
        if original_id is not None:
            original_post = original_content_creator.id_to_post[original_id]
            post_index = original_content_creator.posts.index(original_post)
            if mutated_post.get("_delete", None):
                # deleting posts
                del original_content_creator.id_to_post[original_id]
                original_content_creator.posts.remove(original_post)

                key = f"posts.{post_index}"
                removed[key] = True
            else:
                # updating value
                if mutated_post.get("value", None):
                    original_post.value = mutated_post["value"]
                    key = f"posts.{post_index}.value"
                    updated.setdefault(key, []).append(mutated_post["value"])

                # only allow updating mentions on existing posts
                for mention in mutated_post.get("mentions", []):
                    mention_id = mention.get("_id", None)
                    if mention_id:
                        original_mention = original_post.id_to_mention[mention_id]
                        mention_index = original_post.mentions.index(original_mention)
                        if mention.get("_delete", None):
                            # deleting mention
                            del original_post.id_to_mention[mention_id]
                            original_post.mentions.remove(original_mention)

                            key = f"posts.{post_index}.mentions.{mention_index}"
                            removed[key] = True
                        else:
                            # updating mention
                            original_mention.text = mention["text"]

                            key = f"posts.{post_index}.mentions.{mention_index}.text"
                            updated.setdefault(key, []).append(mention["text"])
                    else:
                        # creating mentions
                        new_mention = Mention(mention)
                        original_post.add_mention(new_mention)

                        key = f"posts.{post_index}.mentions"
                        added.setdefault(key, []).append(mention)
        else:
            # Creating post
            new_post = Post(mutated_post)
            original_content_creator.add_post(new_post)

            key = f"posts"
            added.setdefault(key, []).append(mutated_post)

    if len(updated) > 0:
        update_result[UPDATE] = updated
    if len(added) > 0:
        update_result[ADD] = added
    if len(removed) > 0:
        update_result[REMOVE] = removed
    return json.dumps(update_result)


# Test cases below
original = """
{
"_id": 1,
"name": "Johnny Content Creator",
"posts": [
{
"_id": 2,
"value": "one",
"mentions": []
},
{
"_id": 3,
"value": "two",
"mentions": [
{
"_id": 5,
"text": "apple"
},
{
"_id": 6,
"text": "orange"
}
]
},
{
"_id": 4,
"value": "three",
"mentions": []
}
]
}
"""
mutation_update = """{ "posts": [{"_id": 2, "value": "too"}] } """
mutation_update_text = """{ "posts": [{"_id": 3, "mentions": [ {"_id": 5, "text": "pear"}]}] }"""

mutation_append = """{"posts": [{"value": "four"}] }"""
mutation_append_mentions = """{"posts": [{"_id": 3, "mentions": [{"text": "banana"}]}]}"""

remove_post = """{ "posts": [{"_id": 2, "_delete": true}] }"""
remove_post_mentions = """{ "posts": [{"_id": 3, "mentions": [{"_id": 6, "_delete": true}]}]}"""

print(generate_update_statement(original, mutation_update))
print(generate_update_statement(original, mutation_update_text))

print(generate_update_statement(original, mutation_append))
print(generate_update_statement(original, mutation_append_mentions))

print(generate_update_statement(original, remove_post))
print(generate_update_statement(original, remove_post_mentions))


multi_posts = """
{
"posts": [
{"_id": 2, "value": "too"},
{"value": "four"},
{"_id": 4, "_delete": true}
]
}
"""
print(generate_update_statement(original, multi_posts))
