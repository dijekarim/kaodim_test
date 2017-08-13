# Rails
1. Fix the following snippet. Given a collection of 1000 draft Articles, update all of them to be published

    ```ruby
    Post.where(status: :draft).each { |p| p.update_attributes(status: :published) }
    ```
    `Answer :`
    ```ruby
    Post.where(status: 'draft').update_all(status: 'published')
    ```

1. Given the following model tables

    `users`

    | id | username | email |
    |:-----:|:----:|--|
    | 2 | Leslie | leslie.dunn@mail.com |
    | 3 | Amy | amy.cook@mail.com |
    | 5 | Gavin | gavin.olson@mail.com |
    | 7 | Lucas | lucas.carr@mail.com |
    | 11 | Jennie | jennie.cruz@mail.com |

    `posts`

    | id | user_id | title |
    |:-----:|:----:|--|
    | 10 | 2 | Trump, in Poland, Asks if West Has the 'Will to Survive'|
    | 11 | 3 | Merkel Has to Deal With Trump, the Question Is How |
    | 12 | 2 | New Jersey Transit Train Derails at Pennsylvania Station |
    | 13 | 5 | Faces of Intermarriage, 50 Years After the Loving Case |
    | 14 | 11 | In Milan, Beer, History and, of Course, Fashion |
    | 15 | 3 | Review: Spider-Man (Again) and All That Sticky Kid Stuff|
    | 16 | 7 | Editorial: Showdown in Hamburg |


    `comments`

    | id | post_id | content |
    |:-----:|:----:|---|
    | 10 | 11 | There are various options that can be discussed |
    | 11 | 12 | This is crazy |
    | 12 | 11 | You could almost see her analyze Trump, run through the various scenarios and approaches for     dealing with him |
    | 13 | 15 | Wow!!! Another somewhat positive review of a super hero movie.  |
    | 14 | 16 | This is a very dangerous man who is making the U.S. into a backwater. |
    | 15 | 12 | They had been aware of the problem but had underestimated just how urgently it needed to be    addressed|

    Use ActiveRecord to present the following serialized response. Optimize your code for performance.

    #### RESPONSE

    ```javascript
    {
      "users": [
        {
          id: 2,
          "username": "Leslie",
          "email": "leslie.dunn@mail.com",
          "posts": [
            {
              "id": 10,
              "title": "Trump, in Poland, Asks if West Has the 'Will to Survive'",
              "comments": []
            },
            {
              "id": 12,
              "title": "New Jersey Transit Train Derails at Pennsylvania Station",
              "comments": [
                {
                  "id": 12,
                  "content": "This is crazy"
                },
                ...
              ]
            },
            ...
          ]
        },
        ...
      ]

    }
    ```
    
    `Answers :`

    using jbuilder :
    ```ruby
    json.users User.all.each do |user|
      json.extract! user, :id, :username, :email
      json.posts user.posts.each do |post|
        json.extract! post, :id, :title
        json.comments post.comments.each do |comment|
          json.extract! comment, :id, :content
        end
      end
    end
    ```

# Ruby 

1. What is `self.included`? How would you use it in mixins?
    
    `Answer :`

    Actually we never been used `self.included`. We just used self at method of module, cause at mixins `self` means return object of its own class. But, i'll try to explain it.
    
    The `self.included` function is called when the module is included. It allows `methods` to be executed in the context of the base (where the module is included).

1. Why is it bad to use `rescue` in the following manner? What do you understand about the ruby exception hierarchy?

    ```ruby
    begin
      do_something()
    rescue Exception => e
      puts e
    end
    ```

    `Answer :`

    When we use `rescue Exception` we rescue from everything, including subclasses such as `SyntaxError`, `LoadError`, and `Interrupt`.

    Cause `Exception` is the root of `Ruby's exception hierarchy`. Exception has another child just like this :

    * Exception
      * fatal
      * NoMemoryError
      * ScriptError
        * LoadError
        * NoImplementedError
        * SyntaxError
      * StandardError
        * ArgumentError
        * IndexError
      * etc.
    
    This exception tree has been explained on ruby's official page.


 # SQL

1. A cooperation booth sells gloves in singles and pairs. Each transaction may be a purchase of a single side of glove, or a combination of left/right gloves. It is invalid, however, when a combination of right-right or left-left gloves to be sold.

    Given the following data, write a SQL query to list the duplicate gloves (for both left-left and right-right) sold.

    Transaction

    | id | sold_at |
    |:--:|:--:|
    | 1 | '2017-05-07 15:29:36 +0800' |
    | 2 | '2017-05-09 17:01:59 +0800' |
    | 3 | '2017-06-10 09:32:01 +0800'|
    | 4 | '2017-06-12 09:34:47 +0800'|


    Gloves

    | id | transaction_id | glove_type |
    |:-----:|:----:|:----:|
    | 10 | 1 | "right" |
    | 11 | 1 | "left" |
    | 12 | 2 | "left" |
    | 13 | 3 | "left" |
    | 14 | 3 | "left" |
    | 15 | 4 | "right" |
    | 16 | 5 | "left" |
    | 17 | 4 | "left" |
    | 18 | 5 | "left" |
    | 19 | 6 | "right" |

    Your query result should be:

    | id | transaction_id | glove_type |
    |:-----:|:----:|:----:|
    | 14 | 3 | "left" |
    | 18 | 5 | "left" |

    `Answer :`


    ```sql
    -- drop function if exist
    drop function get_object_fields();
    drop type result_type;

    -- create result type
    CREATE TYPE result_type AS (
      transaction_id   int,
      glove_types varchar(32)
    );

    -- init function
    CREATE OR REPLACE FUNCTION get_object_fields()
    RETURNS setof result_type 
    AS $$

    DECLARE
      trans RECORD;
      glove_count int;
      data_glove gloves;
      result result_type;
    BEGIN 
      FOR trans in SELECT * FROM transacts LOOP
        execute 'SELECT COUNT(*) FROM gloves WHERE transact_id = $1'
        INTO glove_count
        USING trans.id;
		
      -- check for transactions gloves more than 1
      IF glove_count > 1 THEN
      
        -- get 2 data gloves
        execute 'SELECT glove_type FROM gloves WHERE transact_id = $1 ORDER BY id ASC LIMIT 2'
        INTO data_glove
        USING trans.id;
        
        -- compare glove 1 and glove 2 are same?
        IF data_glove[0] = data_glove[1] THEN
          result.transaction_id := trans.id;
          result.glove_types := data_glove[0];
          RETURN NEXT result;
        END IF;
      END IF;
    END LOOP;
    END $$ 
    LANGUAGE plpgsql; 
    ```
