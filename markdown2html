#!/usr/bin/env python
import sys
import argparse


def parse_args():
    ''' Parses command line argument. '''
    help_text = 'converts markdown to html'
    parser = argparse.ArgumentParser(description=help_text)
    parser.add_argument('filename',
                        action='store',
                        default='-',
                        type=str,
                        nargs='?',
                        help='Input markdown filename. \'-\' for stdin.')
    return parser.parse_args()


SYM_TITLE = '#'
SYM_SUBTITLE = '##'
SYM_TRIPLE_TICK = '```'
SYM_BEGIN_LINK_IMAGE = '!'
SYM_BEGIN_LINK = '['
SYM_END_LINK = ']'
SYM_BEGIN_REF = '('
SYM_END_REF = ')'

TICK_STATE_BEGIN = 1
TICK_STATE_END = 2

TEMPLATE_LINK = '<a href="{ref}">{text}</a>'
TEMPLATE_IMAGE = '<img alt="{text}" src="{ref}"></img>'
TEMPLATE_TITLE = '<h1>{text}</h1>'
TEMPLATE_SUBTITLE = '<h2>{text}</h2>'
TEMPLATE_TICK_BEGIN = '<pre>'
TEMPLATE_TICK_END = '</pre>'
TEMPLATE_LINE = '<p>{text}</p>'


class LinkParser:

    def __init__(self, line, begin_link, end_link, begin_ref, end_ref, is_image):
        self.line = line
        self.begin_link = begin_link
        self.end_link = end_link
        self.begin_ref = begin_ref
        self.end_ref = end_ref
        self.is_image = is_image

    @classmethod
    def find(cls, line):
        state = SYM_BEGIN_LINK
        begin_link = None
        end_link = None
        begin_ref = None
        end_ref = None
        is_image = False
        for i, c in enumerate(line):
            if state == SYM_BEGIN_LINK and c == SYM_BEGIN_LINK:
                begin_link = i
                state = SYM_END_LINK
            elif state == SYM_END_LINK and c == SYM_END_LINK:
                end_link = i
                state = SYM_BEGIN_REF
            elif state == SYM_BEGIN_REF and c == SYM_BEGIN_REF:
                begin_ref = i
                state = SYM_END_REF
            elif state == SYM_END_REF and c == SYM_END_REF:
                end_ref = i
                # detect if there is an '!' right before the '[', then we'll parse this as an "image" tag
                if line[begin_link-1] == '!':
                    is_image = True
                return cls(line, begin_link, end_link, begin_ref, end_ref, is_image)
        return None

    @property
    def result(self):
        text = self.line[self.begin_link+1:self.end_link]
        ref = self.line[self.begin_ref+1:self.end_ref]
        tag = (TEMPLATE_IMAGE.format(ref=ref, text=text)
                if self.is_image else 
                TEMPLATE_LINK.format(ref=ref, text=text))
        # cut out the '!' before the image tag if it exists.
        offset_if_image = (1 if self.is_image else 0)
        ret = [
            self.line[:self.begin_link - offset_if_image],
            tag,
            self.line[self.end_ref+1:],
        ]
        ret = ''.join(ret)
        return ret

    @classmethod
    def replace(cls, line):
        while cls.find(line):
            line = cls.find(line).result
        return line


def process_line(line, tick_state):

    if line.startswith(SYM_TRIPLE_TICK):

        if tick_state == TICK_STATE_BEGIN:
            _, text = line.split(SYM_TRIPLE_TICK, 1)
            line = TEMPLATE_TICK_BEGIN + text.strip() + '\n'
            tick_state = TICK_STATE_END

        elif tick_state == TICK_STATE_END:
            _, text = line.split(SYM_TRIPLE_TICK, 1)
            line = TEMPLATE_TICK_END + text.strip() + '\n'
            tick_state = TICK_STATE_BEGIN

    elif tick_state == TICK_STATE_END:

        pass  # don't format anything inside the ticks

    elif line.startswith(SYM_SUBTITLE):
        _, text = line.split(SYM_SUBTITLE, 1)
        line = TEMPLATE_SUBTITLE.format(text=text.strip()) + '\n'

    elif line.startswith(SYM_TITLE):
        _, text = line.split(SYM_TITLE, 1)
        line = TEMPLATE_TITLE.format(text=text.strip()) + '\n'

    else:
        if len(line.strip()) == 0:
            pass  # ignore a pure-whitespace line
        else:
            line = TEMPLATE_LINE.format(text=line.strip()) + '\n'

    # only replace links if we're not inside the ticks
    if tick_state == TICK_STATE_BEGIN:
        line = LinkParser.replace(line)

    return line, tick_state


def process_file(file):
    tick_state = TICK_STATE_BEGIN
    for line in file:
        line, tick_state = process_line(line, tick_state)
        sys.stdout.write(line)


def main():
    args = parse_args()
    file = open(args.filename, 'r') if args.filename != '-' else sys.stdin
    process_file(file)


def test_replace_links():
    test_input = "Hello [google]  (http://www.google.com/)"
    expected = 'Hello <a href="http://www.google.com/">google</a>'
    result = LinkParser.replace(test_input)
    assert result == expected, "'%s' != '%s'" % (result, expected)


def test():
    test_replace_links()


if __name__ == '__main__':
    main()
